---
title: "Getting started with Nextflow"
teaching: 40
exercises: 10
questions:
- "What is the shell?"
- "What is a command shell"
- "Why would I use one?"
objectives:
- "Explain how the shell relates to the keyboard, the screen, the operating system, and users’ programs."
- "Explain when and why command-line interfaces should be used instead of graphical interfaces."
keypoints:
- "A shell is a program whose primary purpose is to read commands and run other programs."
- "This lesson uses Bash, the default shell in many implementations of Unix."
- "Programs can be run in Bash by entering commands at the command-line prompt."
- "The shell’s main advantages are its high action-to-keystroke ratio, its support for automating repetitive tasks, and its capacity to access networked machines."
- "A significant challenge when using the shell can be knowing what commands need to be run and how to run them."
---

## Background
Humans and computers commonly interact in many different ways, such as through a keyboard and mouse, touch screen interfaces, or using speech recognition systems. The most widely used way to interact with personal computers is called a graphical user interface (GUI). With a GUI, we give instructions by clicking a mouse and using menu-driven interactions.

While the visual aid of a GUI makes it intuitive to learn, this way of delivering instructions to a computer scales very poorly. Imagine the following task: for a literature search, you have to copy the third line of one thousand text files in one thousand different directories and paste it into a single file. Using a GUI, you would not only be clicking at your desk for several hours, but you could potentially also commit an error in the process of completing this repetitive task. This is where we take advantage of the Unix shell. The Unix shell is both a command-line interface (CLI) and a scripting language, allowing such repetitive tasks to be done automatically and fast. With the proper commands, the shell can repeat tasks with or without some modification as many times as we want. Using the shell, the task in the literature example can be accomplished in seconds

## The Shell
The shell is a program where users can type commands. With the shell, it’s possible to invoke complicated programs like climate modeling software or simple commands that create an empty directory with only one line of code. The most popular Unix shell is Bash (the Bourne Again SHell — so-called because it’s derived from a shell written by Stephen Bourne). Bash is the default shell on most modern implementations of Unix and in most packages that provide Unix-like tools for Windows. Note that ‘Git Bash’ is a piece of software that enables Windows users to use a Bash like interface when interacting with Git.

Using the shell will take some effort and some time to learn. While a GUI presents you with choices to select, CLI choices are not automatically presented to you, so you must learn a few commands like new vocabulary in a language you’re studying. However, unlike a spoken language, a small number of “words” (i.e. commands) gets you a long way, and we’ll cover those essential few today.

The grammar of a shell allows you to combine existing tools into powerful pipelines and handle large volumes of data automatically. Sequences of commands can be written into a script, improving the reproducibility of workflows.

In addition, the command line is often the easiest way to interact with remote machines and supercomputers. Familiarity with the shell is near essential to run a variety of specialized tools and resources including high-performance computing systems. As clusters and cloud computing systems become more popular for scientific data crunching, being able to interact with the shell is becoming a necessary skill. We can build on the command-line skills covered here to tackle a wide range of scientific questions and computational challenges.

Let’s get started.

When the shell is first opened, you are presented with a prompt, indicating that the shell is waiting for input.

```bash
$ 
```

The prompt might even include more than this. Do not worry if your prompt is not just a short $. This lesson does not depend on this additional information and it should also not get in your way. The only important item to focus on is the $ character itself and we will see later why.

So let’s try our first command, ls, which is short for listing. This command will list the contents of the current directory:

```bash
$ ls
```

```
shell-lesson-data
```
{: .output}

If the shell can’t find a program whose name is the command you typed, it will print an error message such as:

```bash
$ ks
```

```
ks: command not found
```
{: .output}

This might happen if the command was mis-typed or if the program corresponding to that command is not installed.
