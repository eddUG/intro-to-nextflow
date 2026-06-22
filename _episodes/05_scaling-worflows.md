---
title: "Understanding a production Genomics pipeline"
teaching: 50
exercises: 10
questions:
- "How is a production Nextflow pipeline organized?"
- "How do workflows, modules and configuration files fit together?"
- "How does the MalariaGEN SNP genotyping pipeline map to the workflow concepts we have learned?"
- "What changes when moving from a laptop to an HPC cluster?"
- "How can I navigate and run a real DSL2 pipeline?"

objectives:
- "Interpret the structure of a production DSL2 repository."
- "Identify the role of `main.nf`, `workflows`, `modules`, and `configuration files`."
- "Trace data flow through the MalariaGEN SNP genotyping pipeline."
- "Relate production pipeline components to concepts learned in Sessions 2–4."
- "Understand how execution settings differ between laptop and HPC environments."

keypoints:
- "Production pipelines use the same concepts as simple workflows: channels, processes, workflows, parameters, and configs."
- "`main.nf` orchestrates high-level workflow execution."
- "Workflows combine multiple modules into reusable analysis stages."
- "Configuration files separate workflow logic from execution settings."
- "The same workflow can run on a laptop or HPC by changing profiles and executors."
---

## Session 5: Understanding a Production Genomics Pipeline

In Sessions 1–4 we built our own toy whole-genome sequencing workflow. 

Today we will examine a real production pipeline: MalariaGEN Alignment & SNP Genotyping pipeline

Our goal is not to understand every line of code.
Our goal is to recognize familiar concepts operating at a larger scale.

---

### Recap: The Journey So Far
"[Session 2 ]({{ page.root }}/02-working-with-data)"

We learned how to structure sequencing data.

```text
  samplesheet
      ↓
  channels
```

---

"[Session 3 ]({{ page.root }}/03_building-workflows)"

We learned how to describe computational work.

```text
  channels
      ↓
  processes
      ↓
  workflows
```

---

"[Session 4 ]({{ page.root }}/04_reproducibility)"

We learned how to make workflows reproducible via:
- parameters
- configs
- profiles
- containers
- resume

---

#### What We Built

Recall our training workflow.

```text
  FASTQ
    ↓
  FastQC
    ↓
  BWA-MEM
    ↓
samtools sort
    ↓
samtools index
```

This workflow contained:
- channels,
- processes,
- workflow blocks,
- parameters,
- configuration files.

---

#### What we will explore in this session

The MalariaGEN SNP genotyping pipeline contains the same ideas.

It simply contains:
- More modules
- More workflows
- More QC
- More automation

---

### Why study a production pipeline?

In the previous sessions, we built a small workflow from scratch. That workflow was intentionally simple so that we could focus on learning Nextflow concepts.

However, most researchers do not spend their time writing pipelines from scratch. More commonly, they:
- run existing pipelines
- adapt pipelines to local infrastructure
- troubleshoot failed analyses
- change parameters and references
- add or remove workflow stages
- interpret outputs

The goal of today's lesson is therefore not to memorize the MalariaGEN SNP genotyping pipeline. Instead, the goal is to learn how to navigate and reason about a large production workflow using concepts that you already understand.

> ## Discussion
>
> Think about a bioinformatics analysis you have run previously.
> Did you build the workflow yourself, or did you use an existing workflow developed by someone else?
> What modifications did you have to make before you could run it successfully?
{: .discussion}

---

### Pipeline Architecture

The top-level workflow is defined in: `main.nf`

Conceptually:

```text
                     main.nf
                         |
        +----------------+----------------+
        |                                 |
        v                                 v

     MAPPING                     MAPPING_STATS
        |
        v

    mapped BAMs
        |
        v

   SNP_GENOTYPING
        |
        +-----------+
        |           |
        v           v

      VCF      Callable loci
        |
        v

      Zarr
```

> ## Discussion
>
> How does this differ from the workflow we built in Session 3?
> How is it similar? 
{: .discussion}

---

### Where should we start?

When faced with a new repository, many people immediately begin opening random files.

This approach rarely works.

Instead, experienced Nextflow users typically begin by identifying:

- the entry point
- the major workflows
- the configuration files
- the expected inputs and outputs

This provides a mental map of the pipeline before examining implementation details.

For most DSL2 repositories, the entry point is:
`main.nf`

Just as a scientific paper is easier to understand if you first read the abstract, a pipeline is easier to understand if you first read the top-level workflow.

---

### Demo 1: Exploring main.nf

Open:

less `main.nf`

---

#### From training workflow to production workflow

Recall the workflow we built during Session 3:

```text
FASTQ -> FastQC -> BWA -> Sort -> Index
```

This workflow performed alignment and generated indexed BAM files.

The MalariaGEN pipeline performs exactly the same task, but it also incorporates additional quality-control and post-processing steps that are necessary for large-scale population genomics analyses.

As a result, what appeared as a single "alignment stage" in our training workflow becomes an entire workflow in the production pipeline.

---

### Demo 2: Understanding the Mapping workflow

Open:

less `workflows/mapping.nf`

---

### Mapping workflow structure

Conceptually:

```text
  Input BAM
     |
     v
  bam_to_fastq
     |
     v
  FASTQs
     |
     v
read_alignment
     |
     v
alignment_post_processing
     |
     v
mark_duplicates
     |
     v
indel_realigner
     |
     v
fix_mate_information
     |
     v
  mapped BAM
```

---

### Looking for familiar concepts

One common misconception could be that production pipelines use completely different Nextflow features from those encountered in training materials.

In reality, the same small collection of channel operations appears repeatedly in production workflows.

As we inspect the code, look for familiar operations such as:
- `map()`
- `join()`
- `concat()`
- `filter()`

These are the same operations we used when grouping lane-level FASTQ files and constructing channels from the samplesheet.

Recognizing familiar patterns in unfamiliar code is an important skill for working with large pipelines.

### Channel Operations in Production

Observe:

- `join()`
- `map()`
- `concat()`

---

#### Exercise 1

Find one example of:

- `join()`
- `map()`
- `concat()`

Explain what the operation is doing.

---

### From Alignment to biological interpretation

Up to this point, the workflow has focused on preparing sequencing reads for downstream analysis.

The purpose of the genotyping workflow is different.

Rather than transforming sequencing data into aligned reads, it transforms aligned reads into analysis-ready data:

```text
  Aligned Reads
        ↓
  Variant Calls
        ↓
  Analysis-ready genomic data
```

This transition—from sequencing data -> analysis-ready data is where much of the scientific value of the pipeline is generated

---

### Demo 3: Understanding the genotyping workflow

Open:

less `workflows/genotyping.nf`

---

#### Workflow inputs

Observe:

take:
    mapped_bams
    reference_genome
    alleles_vcf

---

#### Variant Calling pipeline

Conceptually:

```text
  Mapped BAMs
      |
      v
  Unified Genotyper
      |
      v
     VCF
      |
      v
   VCF Index
      |
      v
   VCF → Zarr
```

> ## Discussion
>
> What is the major input to the genotyping workflow?
> What is the major output?
{: .discussion}

---

#### Channel Transformations

Observe:

- `.map {`
- `.filter {`

---

#### Exercise 2

Trace the path of one sample through the genotyping workflow. What files are produced?

---

### Separating logic from execution

During Session 4, we discussed one of the most important design principles in modern workflow systems:

Workflow logic should be separated from workflow execution.

This principle becomes increasingly important as workflows become larger and are shared among multiple institutions.

Imagine a workflow that must run:

- on a laptop
- on a university cluster,
- on cloud infrastructure,
- and at CRID.

The analysis logic should remain unchanged.

Only the execution settings should change.

Configuration files allow us to achieve this separation.

### Demo 4: Understanding Configuration files

Open:

less `snp_genotyping_vector.gambiae.config`

---

#### Parameters

Observe:

`params {`

---

#### Profiles

Observe:

`profiles {`

---

> ## Discussion
>
> Why might a laptop use Docker while a cluster uses Singularity?
{: .discussion}

---

### Process Resources

Observe:

`withName:read_alignment`

---

#### Exercise 3

Find one process that requests:
- CPUs
- memory
- container image

---

### Running a production workflow

After understanding the repository structure, the next question is:

- How do we actually run the pipeline?

Most production workflows are launched using a single command that combines:

- the workflow entry point,
- a configuration file,
- one or more parameters,
- and an execution profile.

By this stage of the course, every component of the command should be familiar.

The challenge is no longer understanding the syntax — it is understanding how the pieces fit together.

### Demo 5: Running the Pipeline

Examine a typical command:

``` bash
nextflow run main.nf \
    -c snp_genotyping_vector.gambiae.config \
    --input_manifest_fastq samples.tsv \
    -profile lsf
```

> ## Discussion
>
> Which concepts from Sessions 3 and 4 can you identify in this command?
{: .discussion}

---

### Understanding pipeline outputs

A common mistake among new workflow users is to focus exclusively on successful execution.

In practice, the outputs are usually more important than the workflow itself.

Before running a production workflow, it is helpful to know:

- which files will be produced,
- where they will be written,
- which files are intended for downstream analysis,
- which files are intended for quality control,
- and which files are primarily useful for troubleshooting.

For the MalariaGEN SNP genotyping pipeline, major outputs include:

- Mapped BAMs
- VCFs
- Callable loci
- Zarr datasets
- QC reports

As users and future maintainers of the pipeline, you should become comfortable identifying where these outputs are generated and how they are used in subsequent analyses.

---

#### Exercise 4: Repository hunt

Locate:

- Pipeline entry point
- Alignment workflow
- Genotyping workflow
- Configuration file
- A container definition
- A executor definition

---

#### Exercise 5: Architecture interpretation

Using the diagrams from tHISs session: Draw the path taken by a FASTQ file from input to final variant output.

Include:
- Alignment
- Mapping statistics
- Genotyping
- VCF generation
- Zarr conversion

---

### When things go wrong: troubleshooting a production pipeline

Production workflows rarely run successfully the first time on a new system.

Common issues include:

- missing references,
- incorrect file paths,
- missing containers,
- insufficient resources,
- scheduler configuration problems,
- incompatible software versions.

When a workflow fails, experienced users typically ask:

- Did the workflow start?
- Which process failed?
- What command was executed?
- What do the logs report?
- Was the failure caused by the workflow or the execution environment?

These troubleshooting skills are often more valuable than the ability to write new workflow code.

---

## Summary

In this section, we learned that:
- Production pipelines use the same concepts as simple workflows.
- Workflows combine many modules into reusable analysis stages.
- Configuration files control execution.
- Profiles adapt pipelines to different environments.
- Real genomics pipelines are built from the same building blocks we have already learned.

---
