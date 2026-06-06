---
title: "Building Analysis Workflows"
teaching: 50
exercises: 10
questions:
- "What is a process in Nextflow?"
- "How do I define inputs and outputs for a process?"
- "How do processes receive data from channels?"
- "How can I connect processes into a workflow?"

objectives:
- "Understand the role of processes in Nextflow."
- "Write processes using input, output and script stanzas."
- "Implement FastQC, BWA-MEM, samtools sort and samtools index as processes."
- "Connect multiple processes into a complete workflow."

keypoints:
- "Processes describe computational tasks."
- "Outputs from one process become inputs to another."
- "Complex workflows are built by connecting simple processes."
---

# Session 3: Building Analysis Workflows

## Recap from Session 2

We transformed:

    samplesheet
        ↓
    lane-level tuples
        ↓
    grouped samples
        ↓
    merged FASTQ files

Today, we attach analysis steps to these sample-level inputs.

## What is a Process?

A process describes a **single computational task**.

Examples: - FastQC - BWA-MEM - samtools sort - samtools index

> Channels carry data. Processes perform work.

## Demo 1: FastQC

Goal:

    Merged FASTQs
            ↓
          FastQC
            ↓
      FastQC reports

Create `01_fastqc.nf`.

``` nextflow
nextflow.enable.dsl=2

params.samplesheet = "data/samplesheet/sampleSheet.csv"
params.outdir      = "results/01_fastqc"

process MERGE_LANES {

    tag "${sample_id}"
    publishDir "${params.outdir}/merged_fastq", mode: "copy"

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

process FASTQC {

    tag "${sample_id}"
    publishDir "${params.outdir}/fastqc", mode: "copy"

    input:
    tuple val(sample_id), path(read1), path(read2)

    output:
    tuple val(sample_id),
          path("*_fastqc.html"),
          path("*_fastqc.zip")

    script:
    """
    fastqc ${read1} ${read2}
    """
}

workflow {

    Channel
        .fromPath(params.samplesheet)
        .splitCsv(header: true)
        .map { row ->
            tuple(
                row.sample_id,
                file(row.read1_path, checkIfExists: true),
                file(row.read2_path, checkIfExists: true)
            )
        }
        .groupTuple(by: 0)
        .set { grouped_reads }

    merged_reads = MERGE_LANES(grouped_reads)

    FASTQC(merged_reads)
}
```

Run:

``` bash
nextflow run 01_fastqc.nf
```

Inspect:

``` bash
tree results/
ls work/
```

Discussion: 
- How many FastQC tasks ran? 
- Why?

Discussion: 
- Is there any new syntax here? 
- What changed?

## Exercises

1.  Identify the `tag`, `input`, `output`, and `script` stanzas.
2.  Predict how many FastQC tasks will run.
3.  Modify the FastQC tag to include the process name.
4.  Draw the workflow diagram.
5.  Match each output file to the process that generates it.
6.  Explain what information flows between successive processes.


# Connecting Processes Together

In the previous section, we introduced the concept of a **process** and implemented our first analysis step using FastQC.

The FastQC process operated independently on each sample and produced quality control reports.

Most bioinformatics analyses, however, require **multiple computational steps**.

This raises an important question:

> How do we connect multiple processes together?

The answer lies in **passing outputs from one process as inputs to another**.

Conceptually, this looks like:

```text
Input data
    ↓
Process A
    ↓
Output A
    ↓
Process B
    ↓
Output B
```

In Nextflow, the outputs emitted by one process become channels that can be consumed by downstream processes.



## Demo 2: Alignment with BWA-MEM

The next step in our workflow is to align sequencing reads to a reference genome.

Alignment identifies where each sequencing read originated within the genome.

Conceptually:

```text
Merged FASTQs
        ↓
     BWA-MEM
        ↓
        SAM
```

---

### What is a SAM file?

SAM stands for **Sequence Alignment/Map**.

A SAM file contains information describing:

- which reference sequence a read aligned to,
- where the alignment occurred,
- the quality of the alignment,
- properties of the read pair.

SAM files are text files and can become very large.

---

Create `02_bwa.nf`.

We will reuse many ideas introduced in Demo 1.

``` nextflow
nextflow.enable.dsl=2

params.samplesheet = "data/samplesheet/sampleSheet.csv"
params.reference   = "data/reference/ref.fa"
params.outdir      = "results/02_bwa"

process MERGE_LANES {

    tag "${sample_id}"
    publishDir "${params.outdir}/merged_fastq", mode: "copy"

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

process BWA_MEM {

    tag "${sample_id}"
    publishDir "${params.outdir}/sam", mode: "copy"

    input:
    tuple val(sample_id), path(read1), path(read2)
    path reference

    output:
    tuple val(sample_id), path("${sample_id}.sam")

    script:
    """
    bwa mem ${reference} ${read1} ${read2} > ${sample_id}.sam
    """
}

workflow {

    Channel
        .fromPath(params.samplesheet)
        .splitCsv(header: true)
        .map { row ->
            tuple(
                row.sample_id,
                file(row.read1_path, checkIfExists: true),
                file(row.read2_path, checkIfExists: true)
            )
        }
        .groupTuple(by: 0)
        .set { grouped_reads }

    merged_reads = MERGE_LANES(grouped_reads)

    BWA_MEM(merged_reads, file(params.reference, checkIfExists: true))
}
```

New concept: - reference genome input

Run:

``` bash
nextflow run 02_bwa.nf
```

Inspect:

``` bash
head sample.sam
```

---
> ## Discussion
>
> Which inputs vary from sample to sample?
>
> Which inputs remain constant for all samples?
{: .discussion}
---

> ## What do you observe?
>
> The SAM file contains many columns.
>
> Each row corresponds to an aligned sequencing read.
{: .callout}

---

## Discussion

Suppose the workflow receives five samples.

How many BWA tasks will execute?

Explain your reasoning.

---

Discussion: 
- What information does a SAM file contain? 
- How many BWA tasks ran?

> ## Key concept
>
> The structure of the input channel continues to determine workflow execution.
{: .callout}

## Demo 3: Sorting Alignments

SAM files are text-based and relatively inefficient for downstream analysis.

Most tools expect alignments stored as **sorted BAM files**.

Conceptually:

```text
SAM
 ↓
samtools sort
 ↓
BAM
```

## What is a BAM file?

BAM stands for **Binary Alignment/Map**.

Compared with SAM files, BAM files are:

- compressed,
- smaller,
- faster to process.

Many downstream tools require BAM files to be sorted by genomic coordinate.


Create `03_sort.nf`.

We will extend the workflow developed in Demo 2.

``` nextflow
nextflow.enable.dsl=2

params.samplesheet = "data/samplesheet/sampleSheet.csv"
params.reference   = "data/reference/ref.fa"
params.outdir      = "results/03_sort"

process MERGE_LANES {

    tag "${sample_id}"
    publishDir "${params.outdir}/merged_fastq", mode: "copy"

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

process BWA_MEM {

    tag "${sample_id}"
    publishDir "${params.outdir}/sam", mode: "copy"

    input:
    tuple val(sample_id), path(read1), path(read2)
    path reference

    output:
    tuple val(sample_id), path("${sample_id}.sam")

    script:
    """
    bwa mem ${reference} ${read1} ${read2} > ${sample_id}.sam
    """
}

process SAMTOOLS_SORT {

    tag "${sample_id}"
    publishDir "${params.outdir}/bam", mode: "copy"

    input:
    tuple val(sample_id), path(sam)

    output:
    tuple val(sample_id), path("${sample_id}.bam")

    script:
    """
    samtools sort -o ${sample_id}.bam ${sam}
    """
}

workflow {

    Channel
        .fromPath(params.samplesheet)
        .splitCsv(header: true)
        .map { row ->
            tuple(
                row.sample_id,
                file(row.read1_path, checkIfExists: true),
                file(row.read2_path, checkIfExists: true)
            )
        }
        .groupTuple(by: 0)
        .set { grouped_reads }

    merged_reads = MERGE_LANES(grouped_reads)

    aligned = BWA_MEM(merged_reads, file(params.reference, checkIfExists: true))

    SAMTOOLS_SORT(aligned)
}
```

Run:

``` bash
nextflow run 03_sort.nf
```

## Inspecting Results

Locate the BAM files.

Inspect alignments:
```bash
samtools view sample.bam | head
```

> ## What do you observe?
>
> The BAM file cannot be opened directly with a text editor.
>
> `samtools view` allows us to inspect its contents.
{: .callout}


Discussion: 
- Why do we sort BAM files?

## Discussion

Why do we sort alignments?

Possible answers include:

- downstream requirements,
- efficient access,
- compatibility with indexing.


> ## Key concept
>
> Outputs from one process can become inputs to another process.
>
> Complex workflows emerge by chaining simple tasks together.
{: .callout}

## Exercises

### Exercise 1

Identify all inputs required by the BWA process.

Classify them as:

- sample-specific,
- shared across all samples.

---

### Exercise 2

What information is passed from BWA to SAMTOOLS_SORT?

---

### Exercise 3

If the BWA process produces outputs for five samples, how many sorting tasks will execute?

Explain your reasoning.

---

### Exercise 4

Describe one advantage of BAM files compared with SAM files.

---

## Summary

In this section, we learned that:

- processes can consume outputs produced by upstream processes,
- BWA-MEM aligns sequencing reads to a reference genome,
- SAM files contain alignment information,
- samtools sort converts SAM files into sorted BAM files,
- workflow complexity arises through the composition of simple processes.

In the next section, we will generate BAM index files and assemble all of the processes developed so far into a complete workflow.

## Demo 4: Indexing BAM Files

The final analytical step in our toy workflow is to generate **BAM index files**.

Conceptually:

```text
BAM
 ↓
samtools index
 ↓
BAI
```

---

## Why do we need BAM indexes?

Imagine a BAM file containing alignments across the entire *Anopheles gambiae* genome.

Suppose we want to inspect alignments around a single gene.

Without an index:

```text
Start at the beginning
        ↓
Read the entire BAM file
        ↓
Find the region of interest
```

With an index:

```text
Use the index
        ↓
Jump directly to the region of interest
```

Indexes therefore make downstream analyses much more efficient.

---

> ## Examples of tools requiring indexed BAM files
>
> - IGV (Interactive Genomics Viewer)
> - samtools view for genomic regions
> - variant callers
> - coverage analysis tools
{: .callout}

---

Create `04_index.nf`.

We will extend the workflow developed previously.

``` nextflow
nextflow.enable.dsl=2

params.samplesheet = "data/samplesheet/sampleSheet.csv"
params.reference   = "data/reference/ref.fa"
params.outdir      = "results/04_index"

process MERGE_LANES {

    tag "${sample_id}"
    publishDir "${params.outdir}/merged_fastq", mode: "copy"

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

process BWA_MEM {

    tag "${sample_id}"
    publishDir "${params.outdir}/sam", mode: "copy"

    input:
    tuple val(sample_id), path(read1), path(read2)
    path reference

    output:
    tuple val(sample_id), path("${sample_id}.sam")

    script:
    """
    bwa mem ${reference} ${read1} ${read2} > ${sample_id}.sam
    """
}

process SAMTOOLS_SORT {

    tag "${sample_id}"
    publishDir "${params.outdir}/bam", mode: "copy"

    input:
    tuple val(sample_id), path(sam)

    output:
    tuple val(sample_id), path("${sample_id}.bam")

    script:
    """
    samtools sort -o ${sample_id}.bam ${sam}
    """
}

process SAMTOOLS_INDEX {

    tag "${sample_id}"
    publishDir "${params.outdir}/bam", mode: "copy"

    input:
    tuple val(sample_id), path(bam)

    output:
    tuple val(sample_id),
          path(bam),
          path("${bam}.bai")

    script:
    """
    samtools index ${bam}
    """
}

workflow {

    Channel
        .fromPath(params.samplesheet)
        .splitCsv(header: true)
        .map { row ->
            tuple(
                row.sample_id,
                file(row.read1_path, checkIfExists: true),
                file(row.read2_path, checkIfExists: true)
            )
        }
        .groupTuple(by: 0)
        .set { grouped_reads }

    merged_reads = MERGE_LANES(grouped_reads)

    aligned = BWA_MEM(merged_reads, file(params.reference, checkIfExists: true))

    sorted = SAMTOOLS_SORT(aligned)

    SAMTOOLS_INDEX(sorted)
}
```

Run:

``` bash
nextflow run 04_index.nf
```

> ## Discussion
>
> Which process generated this BAM file?
>
> What information is flowing into SAMTOOLS_INDEX?
{: .discussion}

Discussion: 
- What is the purpose of a BAM index?

# Inspecting Results

Locate the generated files:

```bash
tree results/
```

---

> ## What do you observe?
>
> For every BAM file, there is now a corresponding BAI file.
{: .callout}

---

# Discussion

Suppose we processed:

```text
5 samples
```

How many indexing tasks executed?

Why?

---

> ## Key concept
>
> Processes remain independent.
>
> Each sample continues to move through the workflow separately.
{: .callout}

---

# Bringing Everything Together

Over the course of this session, we have built four independent analyses.

Let's revisit the journey.

---

## Step 1: Quality Control

```text
Merged FASTQs
        ↓
      FastQC
        ↓
 FastQC reports
```

---

## Step 2: Alignment

```text
Merged FASTQs
        ↓
     BWA-MEM
        ↓
        SAM
```

---

## Step 3: Sorting

```text
SAM
 ↓
samtools sort
 ↓
BAM
```

---

## Step 4: Indexing

```text
BAM
 ↓
samtools index
 ↓
BAI
```
---

# Demo 5: Constructing a Workflow

Individually, these processes are useful.

Together, they become a workflow.

Conceptually:

```text
Merged FASTQs
        ↓
      FastQC

Merged FASTQs
        ↓
     BWA-MEM
        ↓
 samtools sort
        ↓
 samtools index
```

Notice that:

- FastQC operates independently,
- the alignment branch forms a linear pipeline.

Create `main.nf`

Reuse the processes developed in the earlier demos.

```nextflow
workflow {

    FASTQC(merged_reads)

    aligned = BWA_MEM(merged_reads, reference_ch)

    sorted = SAMTOOLS_SORT(aligned)

    indexed = SAMTOOLS_INDEX(sorted)
}
```

# Running the Complete Workflow

Execute:

```bash
nextflow run main.nf
```

Use a small subset of samples if necessary.

---

# Inspecting Workflow Execution

Explore:
```bash
ls work/
```

Observe:
- multiple task directories,
- tasks grouped by process.

---

Explore:
```bash
tree results/
```

---

> ## Key concept
>
> The mental model from Session 2 still applies.
>
> Channels determine how many tasks execute.
>
> Processes determine what work is performed.
{: .callout}

---

# Session 3 Exercises

## Exercise 8

Draw the workflow developed during this session.

Label:

- inputs,
- processes,
- outputs.

---

## Exercise 9

Match each file type to the process that produces it.

| File type | Process |
|------------|----------|
| FastQC HTML | |
| SAM | |
| BAM | |
| BAI | |

---

## Exercise 10

Explain the difference between:

```text
SAM
```

and

```text
BAM
```

---

## Exercise 11

Which process requires a reference genome?

Why?

---

## Exercise 12

Suppose a new sample appears in the input channel.

What changes would you need to make to the workflow?

Explain your reasoning.

---

## Exercise 13

Suppose the workflow receives 100 samples.

How many tasks will execute in total?

Assume:

- FastQC,
- BWA,
- samtools sort,
- samtools index

all operate on every sample.

---

# Summary

Today we learned that:

- processes describe computational tasks,
- processes receive inputs and emit outputs,
- outputs from one process become inputs to downstream processes,
- multiple processes can be connected into workflows,
- workflow execution still depends on the structure of the input channels.

---

# Looking Ahead

In Session 4, we will focus on making workflows:

- reproducible,
- portable,
- easier to execute across environments.

We will introduce:

```text
    Parameters
        ↓
Configuration files
        ↓
    Profiles
        ↓
Docker containers
        ↓
      -resume
        ↓
Reports and traces
```

These features will bring our toy workflow closer to the real-world MalariaGEN pipelines that inspired this training.

---

# Key Message

> Channels carry data.
>
> Processes perform work.
>
> Workflows connect them together.
>
> Reproducible analyses emerge from the combination of all three.
````

## Summary

-   Processes define computational tasks.
-   Channels connect processes together.
-   Outputs from one process become inputs to another.
-   Complex workflows are built incrementally from simple components.

> Channels carry data. Processes perform work. Workflows connect them
> together.
