---
layout: post
title:  Easy Debugging for C# in DNN
modified: 2018-04-30 08:00:00
categories: DNN
tags: [DNN, Debug, Local, symlink]
share: true
comments: true
---
When I am developing a DNN Module or even a DNN Platform change I typically configure my development enviornment to output the assemblies in the bin directory or I manually copy over the assemblies and the pdb files. This turns into a very tedious process very quickly when I am trying to rapidly develop or debug DNN Code.

## What Is a SymLink? ##
A SymLink or Symbolic Link is a virtual file or folder which references another file or folder somewhere else. Consider the following example:

You are working in the folders:

* C:\Website\
* C:\Source\MyProject\

Suppose there is a file in "MyProject" that changes quite frequently but is used by another application in C:\Website\ such as a DNN Website. 

A SymLink will tell the operating system when you try and access C:\Website\MyFile.txt it will really point to C:\Source\MyProject\MyFile.txt

## How Do I Use a SymLink? ##
Depending on where you are entering the command (powershell or command prompt) the command will vary slightly

| Command     | Terminal Type  |
|-------------|----------------|
| mklink      | Command Prompt |
| New-SymLink | PowerShell     |

<br />
We are going to focus on using mklink with command prompt

### Single File SymLink ###
If you want to create a SymLink for just one file the command is pretty straight forward:

`mklink <virtual file name> <actual file location>`

* Where you will replace `<virtual file name>` with the name of the file you want to create.
* Then you will replace `<actual file location>` with the file location of the actual file

// TODO - insert screenshot

### Directory SymLink ###
Creating single file SymLinks is really cool, but you may want to create a SymLink for an entire directory. The syntax changes slightly

`mklink /D <virtual folder name> <actual folder location>`

* Where you will replace `<virtual folder name>` with the name of the folder you want to create
* Then you will replace `<actual folder location>` with the actual folder location

// insert screen shot

## Why Use SymLinks in DNN ##
Now that we understand the concept of SymLinks and how to create them, you may ask why use them in DNN? How do they save me time when I am developing a module, theme or a platform change?

### DNN Theme ###
I was recently working on a new DNN Theme using [nvQuickTheme](http://www.nvquicktheme.com/) and the documentation recommended that you develop the theme inside a DNN instance, preferably on a development instance.

I am a big fan of decoupling my source code from a DNN instance so instead of copying my DNN Theme code inside a DNN instance I create a SymLink. The theme appeared inside DNN just as you would expect.

### DNN Development ###
This method applies for both DNN Module or DNN Platform Development

Suppose you are working on a DNN Module, during rapid development you may update your Visual Studio configuration to drop the assemblies in the bin directory of your DNN Development instance. This does have side-effects and in my experience I only bring over what is needed such as the module dll and any additional libs used by the module. 

You can use a file SymLink to copy over just the assemblies you want

* YourModule.dll
* YourModule.pdb

Don't forget to add a SymLink for the pdb files so you can use the debugger tools in Visual Studio