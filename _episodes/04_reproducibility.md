---
title: "Reproducibility and execution environments"
teaching: 50
exercises: 10
questions:
- "How can I avoid hard-coding values into a workflow?"
- "What is a Nextflow configuration file?"
- "What are profiles and why are they useful?"
- "How can containers improve reproducibility?" 
- "How can I monitor and resume workflow execution?"

objectives:
- "Use parameters to make workflows flexible."
- "Move workflow settings into a Nextflow configuration file."
- "Understand the purpose of execution profiles."
- "Run a workflow using Docker."
- "Use -resume to avoid recomputing completed tasks."
- "Generate execution reports and interpret them."

keypoints:
- "Parameters make workflows reusable."
- "Configuration files separate workflow logic from execution settings."
- "Profiles allow the same workflow to run in different environments."
- "Containers improve reproducibility by standardizing software environments."
- "Nextflow records execution metadata that can be used for monitoring and troubleshooting."

---

# Session 4: Making Workflows Reproducible and Portable

### Recap 

In Session 2, we learned how to structure sequencing data.

samplesheet
    ↓
channels

In Session 3, we learned how to describe computational work.

channels
    ↓
processes
    ↓
workflows

Here, we focus on making workflows:
- easier to use
- easier to share
- easier to rerun
- easier to scale

## Why does reproducibility matter?

Suppose you analyse a dataset today.

Six months later, you want to rerun the analysis.

Questions arise:
- Which reference genome did I use?
- Which software versions were installed?
- Which parameters did I choose?
- Can someone else reproduce these results?

These are precisely the challenges that workflow systems help address.

### Demo 1: Introducing parameters

Consider this workflow fragment:

``` nextflow
params.reference = "data/reference/ref.fa"
params.samplesheet = "data/samplesheet/sampleSheet.csv"
params.outdir = "results"
``` 
Run with alternative values:

``` bash
nextflow run main.nf \
    --reference data/reference/An_gambiae.fa \
    --outdir results_test
```

> ## Discussion
>
> What advantages do parameters provide?
{: .discussion}

### Demo 2: Moving settings into nextflow.config


Create: `nextflow.config`

Add:

```
params {
    reference = "data/reference/ref.fa"
    samplesheet = "data/samplesheet/sampleSheet.csv"
    outdir = "results"
}
```

> ### Discussion
>
> Why might separating configuration from workflow code be useful?
> 
{: .discussion}


### Demo 3: Profiles

Profiles allow the same workflow to behave differently depending on where it runs.

---

Add `nextflow.config`.

---

``` 
profiles {

    local {
        process.executor = 'local'
    }

    docker {
        docker.enabled = true
        process.executor = 'local'
    }
}
```

Running with profiles

Local:

``` bash
nextflow run main.nf -profile local
```

Docker:

``` bash
nextflow run main.nf -profile docker
```


> ## Discussion
>
> Why might we want multiple profiles?
> 
{: .discussion}

---

### Demo 4: Introducing Docker

Conceptually:

Workflow
    ↓
Container
    ↓
Identical software environment

---

Adding containers

Modify a process:

```
process FASTQC {

    container 'biocontainers/fastqc:v0.12.1_cv8'

    ...
}
```

Running with Docker

``` bash
nextflow run main.nf -profile docker
``` 

> ## Discussion
>
> What problem do containers solve?
> 
{: .discussion}

---

### Demo 5: Resume functionality

Run:
``` bash
nextflow run main.nf
``` 

Interrupt the workflow.

Re-run:

``` bash
nextflow run main.nf -resume
``` 

> ## Discussion
>
> Why is this useful?
> 
{: .discussion}


### Demo 6: Execution reports

Run:

``` bash
nextflow run main.nf \
    -with-report report.html \
    -with-trace trace.txt \
    -with-timeline timeline.html
``` 

Inspect Report, Trace and timeline

Open`report.html`

head `trace.tx`

Open `timeline.html`



## Exercises

### Exercise 1

Convert hard-coded paths in main.nf into parameters.

---

### Exercise 2

Create a nextflow.config file containing:
- `reference`,
- `samplesheet`,
- `outdir`.

---

### Exercise 3

Predict the effect of:
``` bash
nextflow run main.nf --outdir new_results
``` 

### Exercise 4

Describe one advantage of profiles.

### Exercise 5

Explain the difference between `local` profile and `docker` profile

### Exercise 6

Suppose a workflow failed after completing `FastQC` and `alignment`.

What would happen if you reran it using:

``` bash
nextflow run main.nf -resume
``` 

### Exercise 7

Which execution report would you inspect to determine:

- task runtimes?
- parallel execution patterns?

---

## Summary

In this section, we learned that:

- parameters make workflows flexible,
- configuration files separate settings from logic,
- profiles adapt workflows to different environments,
- containers improve reproducibility,
- `-resume` avoids unnecessary recomputation,
- execution reports help monitor and troubleshoot analyses.

---

### Looking Ahead

In Session 5, we will connect these concepts to the MalariaGEN SNP genotyping workflow.

We will ask:

- Which parts scale?
- What changes on HPC systems?
- How do executors differ?
- How do real production workflows organize modules and configurations?


---

### Key message

- Processes define work.
- Configurations define execution.
- Containers define software environments.
- Together, they enable reproducible and scalable bioinformatics workflows.

---


````
