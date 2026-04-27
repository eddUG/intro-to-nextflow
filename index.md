---
layout: lesson
title: "Introduction to Nextflow for vector genomics"
root: .
---

Welcome to the Nextflow training for vector genomics: an introduction to building and running reproducible bioinformatics workflows.

This workshop introduces the use of **Nextflow** for managing genomic data analysis pipelines. We cannot cover everything in a few sessions. Instead, we aim to provide a strong foundation that will enable you to confidently run and understand workflows, and begin building your own.

Workflow management is fundamental to modern genomics, where analyses involve multiple steps, tools and datasets. These lessons will introduce you to a structured and scalable way to handle such analyses.

> ## Prerequisites
>
> This lesson assumes basic familiarity with:
>
> - using a computer and navigating files and folders  
> - running commands in a terminal (basic Unix shell)  
>
> You do **not** need prior programming or workflow experience.
>
> If you can recognize a file, a folder, and run simple commands, you are ready.
{: .prereq}


## By the end of the workshop, learners will be able to:

* Understand what workflows and workflow management systems are.
* Run Nextflow workflows on local machines.
* Interpret workflow outputs, logs, and execution directories.
* Use samplesheets to drive data analysis workflows.
* Understand how data flows through workflows using channels.
* Transform and structure input data (e.g., grouping sequencing lanes by sample).
* Execute a multi-step genomics workflow (QC, alignment, summarization).
* Re-run workflows efficiently using Nextflow’s resume functionality.
* Understand how workflows scale to larger systems (HPC / cloud).
* Develop a mental model for reproducible and scalable data analysis.

> ## Getting Started
>
> To get started, follow the directions in the "[Setup]({{ page.root }}/setup.html)"
> tab to:
>
> - install Nextflow  
> - prepare your working environment  
> - download training data  
>
> Please complete setup **before the first session**.
{: .callout}

> ## Training Structure
>
> This workshop is organized into progressive sessions:
>
> **Session 1:**  Introduction to workflows and Nextflow. 
>
> **Session 2:**  Working with data in Nextflow: channels, grouping and merging.
>
> **Session 3:**  Building workflows using processes: QC, alignment and BAM processing.
>
> **Session 4:**  Reproducibility and execution environments: parameters, profiles and containers.
>
> **Session 5:**  Scaling workflows and mapping to real-world pipelines (the MalariaGEN SNP genotyping).
{: .callout}

