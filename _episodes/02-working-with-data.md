
---

# Session 2: Working with Data in Nextflow

## Channels, Pairing, Grouping, and Merging

### Learning Objectives

By the end of this session, you will be able to:

* Create channels from a samplesheet
* Understand how data drives process execution
* Represent paired-end reads as structured inputs
* Group lane-level data by `sample_id`
* Merge FASTQ files across lanes into sample-level inputs

---

## Recap from Session 1

In the previous session, we ran several Nextflow scripts and observed:

* A **process runs once per input item**
* The number of executions depends on the data
* A samplesheet can drive workflow execution

In this session, we focus on **how to shape that data**.

---

## From Files to Data Streams

In traditional scripting, you might write a loop like:

```bash
for file in *.fastq.gz; do
    echo $file
done
```

In Nextflow, we don’t write loops. Instead, we describe **data as a stream**, and Nextflow handles iteration automatically.

---

## What is a Channel?

A **channel** is a stream of data items.

A channel can contain:

* file paths
* values
* tuples (structured data)

Each item in a channel triggers **one execution of a process**.

---

## Demo 1: A Simple Channel

Create a file `channel_demo.nf`:

```nextflow
nextflow.enable.dsl=2

workflow {
    Channel.of(1, 2, 3).view()
}
```

Run:

```bash
nextflow run channel_demo.nf
```

### What happens?

* The channel contains three values: 1, 2, 3
* Each value is printed using `.view()`

---

## Working with a Samplesheet

In real workflows, inputs are described using a **samplesheet**.

Example structure:

```csv
sample_id,read1_path,read2_path
sample1,data/fastq/sample1_L001_R1.fastq.gz,data/fastq/sample1_L001_R2.fastq.gz
sample1,data/fastq/sample1_L002_R1.fastq.gz,data/fastq/sample1_L002_R2.fastq.gz
sample1,data/fastq/sample1_L003_R1.fastq.gz,data/fastq/sample1_L003_R2.fastq.gz
```

Each row represents:

> A paired-end sequencing run (lane)

---

## Demo 2: Parsing the Samplesheet

Create `parse_samplesheet.nf`:

```nextflow
nextflow.enable.dsl=2

params.samplesheet = "data/samplesheet/samplesheet.local.csv"

workflow {

    Channel
        .fromPath(params.samplesheet)
        .splitCsv(header: true)
        .map { row ->
            tuple(
                row.sample_id,
                file(row.read1_path),
                file(row.read2_path)
            )
        }
        .view()
}
```

Run:

```bash
nextflow run parse_samplesheet.nf
```

### What do you observe?

* Each row becomes a tuple:

  ```
  (sample_id, read1, read2)
  ```
* Each tuple represents **one lane-level input**

---

## Paired-End Reads

Each row already represents a **paired-end dataset**:

* `read1` → forward reads
* `read2` → reverse reads

In Nextflow, we treat them as a unit:

```nextflow
tuple(sample_id, read1, read2)
```

---

## The Problem: Lane-Level Data

Each `sample_id` appears multiple times:

* sample1 → lane 1
* sample1 → lane 2
* sample1 → lane 3

Currently:

> Nextflow processes each row independently

---

## The Goal: Sample-Level Processing

We want to:

* group all rows for the same sample
* treat them as one unit

---

## Demo 3: Grouping by Sample

Create `group_lanes.nf`:

```nextflow
nextflow.enable.dsl=2

params.samplesheet = "data/samplesheet/samplesheet.local.csv"

workflow {

    Channel
        .fromPath(params.samplesheet)
        .splitCsv(header: true)
        .map { row ->
            tuple(
                row.sample_id,
                file(row.read1_path),
                file(row.read2_path)
            )
        }
        .groupTuple(by: 0)
        .view()
}
```

Run:

```bash
nextflow run group_lanes.nf
```

---

### What changed?

Before grouping:

```
(sample1, R1_L001, R2_L001)
(sample1, R1_L002, R2_L002)
(sample1, R1_L003, R2_L003)
```

After grouping:

```
(sample1, [R1_L001, R1_L002, R1_L003], [R2_L001, R2_L002, R2_L003])
```

---

### Key concept

> One channel item now represents one **sample**, not one lane.

---

## Merging Lanes

Now that we have grouped data, we can merge FASTQ files.

---

## Demo 4: Merge FASTQ Files

Create `merge_lanes.nf`:

```nextflow
nextflow.enable.dsl=2

params.samplesheet = "data/samplesheet/samplesheet.local.csv"

process MERGE_LANES {

    tag "${sample_id}"

    input:
    tuple val(sample_id), path(read1_files), path(read2_files)

    output:
    tuple val(sample_id),
          path("${sample_id}_R1.fastq.gz"),
          path("${sample_id}_R2.fastq.gz")

    script:
    """
    cat ${read1_files.join(' ')} > ${sample_id}_R1.fastq.gz
    cat ${read2_files.join(' ')} > ${sample_id}_R2.fastq.gz
    """
}

workflow {

    Channel
        .fromPath(params.samplesheet)
        .splitCsv(header: true)
        .map { row ->
            tuple(
                row.sample_id,
                file(row.read1_path),
                file(row.read2_path)
            )
        }
        .groupTuple(by: 0)
        .set { grouped_reads }

    MERGE_LANES(grouped_reads)
}
```

Run:

```bash
nextflow run merge_lanes.nf
```

---

## Inspect Results

```bash
ls work/
```

Navigate into a directory:

```bash
cd work/<hash>/
ls
```

---

### What do you observe?

* Fewer process executions (one per sample)
* Output FASTQ files are merged

---

## Execution Behavior

| Stage           | Execution           |
| --------------- | ------------------- |
| Before grouping | one task per lane   |
| After grouping  | one task per sample |

---

## Key Concepts

* Channels carry data
* Each item = one task
* Data structure determines execution
* Grouping changes workflow behavior

---

## Exercises (Optional)

### Exercise 1

Modify `parse_samplesheet.nf` to print only `sample_id`.

---

### Exercise 2

Count how many lanes each sample has.

(Hint: use `.groupTuple()` and inspect list lengths)

---

### Exercise 3

Modify the merge script to print filenames before merging.

---

## Summary

In this lesson, you learned how to:

* transform a CSV into a Nextflow channel
* represent paired-end data
* group lane-level data into sample-level inputs
* merge FASTQ files
* control how workflows execute through data

---

