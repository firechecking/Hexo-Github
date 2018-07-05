---
title: Kaldi文档阅读
date: 2017-02-13 11:44:54
tags: [机器学习,Kaldi,语音识别]
categories: [机器学习,语音识别,Kaldi]
comments: true
---
# Modules
## 官方文档
<http://kaldi-asr.org/doc/index.html>
## Mac OSX 10.11.5下的Kaldi安装与部署教程
<https://changkun.us/archives/2016/06/81/><!--more-->
## 相关文章
* [从声学模型算法总结 2016 年语音识别的重大进步](http://www.leiphone.com/news/201701/ZEDsG4fOpI2zr2OA.html)

## About the Kaldi project
* compile against the OpenFst toolkit
* intended audience is speech recognition researchers or researchers-in-training. In general, Kaldi is not a speech recognition toolkit "for dummies."
* We emphasize generic algorithms and universal recipes
* Each command-line program generally works for a limited set of cases 
* We aim for the toolkit to as loosely coupled as possible. In general this means that any given header should need to #include as few other header files as possible
* The matrix library, in particular, only depends on code in one other subdirectory so it can be used independently of almost all the rest of Kaldi.
* have code and scripts for most standard techniques(standard linear transforms,MMI, boosted MMI and MCE discriminative training,)

## Software required to install and run Kaldi
1. ideal computing environment
	1. a cluster of Linux machines (any major distribution) running ba (SGE)
	1. access to shared directories via NFS or some similar network filesystem
	1. have NVidia GPUs which you can use for neural net training
1. Bare minimum computing environment
	1. Bare minimum computing environment
	1. may have to reduce the number of jobs used in some of the example scripts to avoid exhausting your machine's memory.
	1. Kaldi is best tested on Debian and Red Hat Linux, but will run on any Linux distribution, or on Cygwin or Mac OsX.

## Kaldi tutorial
### Prerequisites
* This tutorial assumes that you know the basics of speech recognition using the HMM-GMM approach.
*  If ATLAS is not already installed as a library on your system you will have to compile it, and this requires that CPU throttling be turned off

### Overview of the distribution
1. The tools/ directory
	* The directory "tools/' is where we install things that Kaldi depends on in various ways
1. The src/ directory
	* If you are having problems with a build process, one solution is to try modifying kaldi.mk by hand.

### Running the example scripts
1. Getting started, and prerequisites.
	* The main file we will be looking at is run.sh. Note: run.sh is not intended to be run directly from the shell; the idea is that you run the commands in it one by one, by hand.
1. Data preparation
	* We will first need to configure whether the jobs need to run locally or on the Oracle GridEngine
	* The next step is to create the test and training sets from the RM corpora
	* not all of them could be read by Kaldi's C++ programs and need to be processed using OpenFST tools before Kaldi can use them.
	* The next step is to create the raw language files that Kaldi uses. In most cases, these will be text files in integer formats.This will create a new folder called lang within the local folder which will contain an FST describing the language in question.
	* The next step is to use the files created in the previous step to create an FST describing the grammar for the language.
1. Feature extraction
	* The next step is to extract the training features
	* The script format(.scp) is a text-only format has lines with a key, and then an "extended filename" that tells Kaldi where to find the data.
	* The archive format(.ark) may be text or binary (you can write in text mode with the ",t" modifier; binary is default). The format is: the key (e.g. utterance id), then a space, then the object data.
1. Monophone training(单音素训练)
	* The next step is to train monophone models.
	* The moniphone system is now finished and we will do triphone training and decoding in the next step of tutorial.

### Reading and modifying the code
1. Common utilities
	* base/kaldi-common.h
	* This #includes a number of things from the base/ directory that are used by almost every Kaldi program.
	* the matrix/ directory only depends on the base/ directory.
1. Matrix library
	* This library is basically a C++ wrapper for BLAS and LAPACK
1. Acoustic modeling code
	* gmm/diag-gmm.h (this class stores a Gaussian Mixture Model)。This is just a single GMM, not a whole collection of GMMs
	* gmm/am-diag-gmm.h; this class stores a collection of GMMs
1. Feature extraction code
	* MfccOptions struct give some idea what kind of options are supported in MFCC feature extraction
1. Acoustic decision-tree and HMM topology code
	* tree/build-tree.h is the main top-level function for building the decision tree. returns a pointer the type EventMap. 
	* EventMap is a type that stores a function from a set of (key, value) pairs to an integer. but the keys represent phonetic-context positions (typically 0, 1 or 2. There is also a special key, -1, that roughly represents the position in the HMM.) and the values represent phones.
	* In hmm/hmm-topology.h. Class HmmTopology defines a set of HMM topologies for a number of phones.(对每个音素的隐马尔科夫矩阵)

## Kaldi for Dummies tutorial
### Introduction
* This is a step by step tutorial for absolute beginners on how to create a simple ASR (Automatic Speech Recognition) system in Kaldi toolkit using your own set of data.
* You will learn how to install Kaldi, how to make it work and how to run an ASR system using your own audio data. As an effect you will get your first speech decoding results.

### Environment
* use Linux(write with the environment Ubuntu 14.10)
* kaldi-trunk - main Kaldi directory which contains:
	* egs – example scripts allowing you to quickly build ASR systems for over 30 popular speech corporas (documentation is attached for each project)
	* misc – additional tools and supplies, not needed for proper Kaldi functionality
	* src – Kaldi source code
	* tools – useful components and external tools
	* windows – tools for running Kaldi using Windows.

### Your exemplary project
* You have some amount of audio data that contain only spoken digits by at least several different speakers.

### Data preparation
1. Audio data
	* a set of 100 files
	* 10 different speakers
	* Task
		* put all of one person's audio files to test folder. Put the rest (9 speakers) into train folder
1. Acoustic data
	* some text files that will allow Kaldi to communicate with your audio data.
	* a text file with some number of strings
1. Language data
### Project finalization
### Running scripts creation
### Getting results

## Examples included with Kaldi