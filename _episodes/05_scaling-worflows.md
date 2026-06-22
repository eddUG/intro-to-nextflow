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

### Demo 1: Exploring main.nf

Open:

less `main.nf`

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

## Summary

In this section, we learned that:
- Production pipelines use the same concepts as simple workflows.
- Workflows combine many modules into reusable analysis stages.
- Configuration files control execution.
- Profiles adapt pipelines to different environments.
- Real genomics pipelines are built from the same building blocks we have already learned.

---
