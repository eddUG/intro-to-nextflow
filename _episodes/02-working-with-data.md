---
title: "Working with Data"
teaching: 50
exercises: 10
questions:
- "What is a channel in Nextflow and how does it control workflow execution?"
- "How do I create a channel from a samplesheet?"
- "How does Nextflow represent paired-end sequencing data?"
- "How can I group multiple inputs (e.g., sequencing lanes) by sample?"
- "How does grouping data affect process execution?"
- "How can I merge multiple FASTQ files within a workflow?"

objectives:
- "Understand how channels are used to pass data between processes."
- "Create channels from structured input files such as a samplesheet."
- "Represent paired-end sequencing data using tuples."
- "Group lane-level data into sample-level inputs using Nextflow operators."
- "Explain how data structure determines how processes are executed."
- "Implement a simple process to merge sequencing lanes."

keypoints:
- "Channels are streams of data that determine how many times a process runs."
- "Each item in a channel triggers one independent execution of a process."
- "Structured data (e.g., tuples) allows multiple related inputs to be passed together."
- "Grouping data changes workflow behavior from lane-level to sample-level processing."
- "The structure of input data directly controls workflow execution."
- "Nextflow enables scalable workflows by automatically parallelizing tasks based on input data."
---

## From Files to Data Streams

In traditional scripting, you might write a loop like:

```bash
for file in *.fastq.gz; do
    echo $file
done
```

In Nextflow, we don’t write loops. Instead, we describe data as a stream and Nextflow handles iteration automatically. 
Instead of explicitly looping over files, we define the data and let Nextflow determine how to process it.
This shift, from writing loops to shaping data, is one of the most important ideas in Nextflow.


## What is a Channel?

A channel is a stream of data items.

A channel can contain:
- file paths
- values
- tuples (structured data)

Each item in a channel triggers one execution of a process.

> You can think of a channel as a queue of inputs flowing through your workflow.

This means:

> the number of times a process runs is determined entirely by the data in the channel.

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

> ## What happens? 
>
> The channel contains three values: 1, 2, 3
> Each value is printed using `.view()`
> 
> ## Note
> The .view() operator is extremely useful for inspecting the contents of a channel during workflow development.
{: .callout}

## Optional: Channels from Files

Channels are often created from file patterns:

```nextflow

workflow {
    Channel.fromPath("data/fastq/*.fastq.gz").view()
}
```
This approach is useful when working directly with files. However, in this training we will use a samplesheet which provides more structure and flexibility.

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

This means:

> Each row corresponds to one independent unit of work.

## Demo 2: Parsing the samplesheet

In this step, we convert the samplesheet into a channel that Nextflow can use.

Create `parse_samplesheet.nf`:

```nextflow
nextflow.enable.dsl=2

params.samplesheet = "data/samplesheet/sampleSheet.csv"

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

> ## What do you observe? 
>
> Each row becomes a tuple:
>  ```
> (sample_id, read1, read2)
>  ```
> Each tuple represents one lane-level input
{: .callout}

This transformation is important because it turns raw metadata into structured inputs that Nextflow can process.

## Paired-end reads

Each row already represents a paired-end dataset:
- `read1` → forward reads
- `read2` → reverse reads

In Nextflow, we treat them as a unit:

```nextflow
tuple(sample_id, read1, read2)
```
This ensures that both files are always processed together in downstream steps.

## The problem: Lane-level data

Each `sample_id` appears multiple times:
- sample1 -> lane 1
- sample1 -> lane 2
- sample1 -> lane 3

Currently:

> Nextflow processes each row independently

This means:

> Each sequencing lane is treated as a separate task

## The goal: sample-level processing

However, in most analyses, we are interested in samples, not lanes.

We want to:

- group all rows for the same sample
- treat them as one unit

This requires restructuring our data before passing it to downstream processes

## Demo 3: Grouping by sample

Create `group_lanes.nf`:

```nextflow
nextflow.enable.dsl=2

params.samplesheet = "data/samplesheet/sampleSheet.csv"

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

> ## What changed? 
>
>  Before grouping:
> ```
> (sample1, R1_L001, R2_L001)
> (sample1, R1_L002, R2_L002)
> (sample1, R1_L003, R2_L003)
> ```
> After grouping:
> ```
> (sample1, [R1_L001, R1_L002, R1_L003], [R2_L001, R2_L002, R2_L003])
> ```
{: .callout}

> ## Key concept
>
> One channel item now represents one sample, not one lane.
{: .callout}

This transformation changes how the workflow executes: instead of running once per lane, it will now run once per sample.

## Merging Lanes

Now that we have grouped data, we can merge FASTQ files.

> Merging combines sequencing data from multiple lanes into a single dataset per sample.

## Demo 4: Merge FASTQ Files

Create `merge_lanes.nf`:

```nextflow
nextflow.enable.dsl=2

params.samplesheet = "data/samplesheet/sampleSheet.csv"

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

## Inspect results

```bash
ls work/
```

Navigate into a directory:

```bash
cd work/<hash>/
ls
```

> ## What do you observe?
>
> Fewer process executions (one per sample)
> Output FASTQ files are merged
{: .callout}


## Execution behavior

| Stage           | Execution           |
| --------------- | ------------------- |
| Before grouping | one task per lane   |
| After grouping  | one task per sample |


> ## Key concept
>
> - Channels carry data
> - Each item = one task
> - Data structure determines execution
> - Grouping changes workflow behavior
{: .callout}

### Exercise 1

Modify `parse_samplesheet.nf` to print only `sample_id`.

### Exercise 2

Count how many lanes each sample has.

(Hint: use `.groupTuple()` and inspect list lengths)

### Exercise 3

Modify the merge script to print filenames before merging.

