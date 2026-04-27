---
layout: page
title: Setup
---

## Setup Instructions

This training introduces **Nextflow**, a workflow management system used in modern bioinformatics.

Please complete all steps **before Session 1** to ensure a smooth experience during the workshop.

## 1. Create your working directory

Open a terminal and run:

```bash
mkdir -p ~/vector-nextflow-training
cd ~/vector-nextflow-training
````

This will be your main working directory for the training.

## 2. Install Java

Nextflow requires **Java 17 or later**.

Check your installation:

```bash
java -version
```

If Java is not installed or is outdated:

### Linux (Ubuntu)

```bash
sudo apt update
sudo apt install openjdk-17-jdk
```

### macOS (Homebrew)

```bash
brew install openjdk@17
```

## 3. Install Nextflow

Download and install Nextflow:

```bash
curl -s https://get.nextflow.io | bash
chmod +x nextflow
mkdir -p ~/bin
mv nextflow ~/bin/
```

Add Nextflow to your PATH:

```bash
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

Test installation:

```bash
nextflow -version
```

## 4. Enable modern Nextflow syntax

We will use Nextflow DSL2.

Enable it:

```bash
export NXF_SYNTAX_PARSER=v2
```

Make it permanent:

```bash
echo 'export NXF_SYNTAX_PARSER=v2' >> ~/.bashrc
source ~/.bashrc
```

## 5. Install Git

Check:

```bash
git --version
```

Install if needed:

### Linux

```bash
sudo apt install git
```

### macOS

```bash
brew install git
```

## 6. Install Visual Studio Code

Download: [https://code.visualstudio.com/](https://code.visualstudio.com/)

Install the following extensions:

- Nextflow
- Docker

Then:

- Open your training folder in VS Code
- Use the integrated terminal

---

## 7. Install Docker

We will run tools using Docker containers.

Check:

```bash
docker --version
```

Test:

```bash
docker run hello-world
```

If Docker is not installed:

- macOS / Windows -> Install **Docker Desktop**
- Linux -> Install **Docker Engine**

## 8. Download training data

Download all training data from:

👉 [https://drive.google.com/drive/folders/1eg9uFvkrouNiF4kv7lnlxD6rz1JVfcV3](https://drive.google.com/drive/folders/1eg9uFvkrouNiF4kv7lnlxD6rz1JVfcV3)

This includes:

- FASTQ files
- Reference genome
- Samplesheet

### Organize your data

Place the files into the following structure:

```bash
~/vector-nextflow-training/data/
```

Expected layout:

```text
data/
├── fastq/
│   ├── *.fastq.gz
├── reference/
│   ├── ref.fa
│   └── ref.fa.fai
└── samplesheet/
    └── sampleSheet.csv
```

### Notes on the samplesheet

- Each row represents one sequencing lane
- The same `sample_id` may appear multiple times
- This will be important during the training

## 9. Final setup check

Run the following commands:

```bash
uname -a
java -version
nextflow --version
git --version
docker --version
echo $NXF_SYNTAX_PARSER
```

## Expected outcome

You should see:

- Java ≥ 17
- Nextflow installed
- Docker working
- `NXF_SYNTAX_PARSER=v2`


## Further reading 

### Nextflow
* [https://training.nextflow.io/latest/hello_nextflow/00_orientation/](https://training.nextflow.io/latest/hello_nextflow/00_orientation/)
* [https://docs.seqera.io/nextflow/install](https://docs.seqera.io/nextflow/install)

### Java
* [https://sdkman.io/](https://sdkman.io/)

### Docker
* [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)

### WSL (Windows users)
* [https://learn.microsoft.com/en-us/windows/wsl/install](https://learn.microsoft.com/en-us/windows/wsl/install)

### VS Code
* [https://code.visualstudio.com/](https://code.visualstudio.com/)

### Nextflow VS Code extension
* [https://marketplace.visualstudio.com/items?itemName=nextflow.nextflow](https://marketplace.visualstudio.com/items?itemName=nextflow.nextflow)

### Git
* [https://git-scm.com/downloads](https://git-scm.com/downloads)





