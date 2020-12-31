---
toc: true
layout: post
description: A relatively simple way to become a code contributor
categories: [contributing, tutorial]
title: Setting up an environment to make Python contributions to the RDKit
---

It has been tricky to contribute code or documentation to the RDKit if you're a Python programmer who doesn't want to deal with the complexities of getting an RDKit build working. We want to make it straightforward for people to contribute, so I'm working on some recipes to make thigs easier. This is the first pass at that.

In order to fix bugs or add features in Python you need to be able to clone a local fork of the RDKit from github, modify the code in that local clone, and then run the local code in order to test it. The problem is that most RDKit functionality requires some binary components that need to be built from C++ and installed in the appropriate places. We're going to work around that problem here by copying the binary components from a recent binary distribution of the RDKit into a local clone of the RDKit repo.

I'm going to explain each of the required steps, but the complete set of steps required is at the bottom of this post. Assuming that you have the prerequisites (explained directly below), I hope that these will "just work" for you, but one never knows... I'd like to be able to include this in the RDKit documentation, so please me know how it goes if you try the recipe out. Please do not add a comment to this blog post, I've created a github issue so that we have the comments in one place. If you don't have a github account, please email me your comments and I'll add them to the issue.

# The steps explained
At the moment this recipe only works on linux and the mac. I will put together a similar recipe for windows and either do a separate post or update this one.

Prerequisites:

- you need to have either anaconda python or miniconda installed and in your path
- you need to have git installed and in your path

You should start by changing into the directory where you want to clone the RDKit source repository and then running:

```
git clone https://github.com/rdkit/rdkit.git
```

That will clone the repo from github into a local directory called rdkit. We now change into that directory and use it to set our RDBASE environment variable:

```
cd rdkit
export RDBASE=`pwd`
```

The next step is to create the conda environment that we're going to use to hold the RDKit binary components and install a recent beta version of the RDKit into that environment:

```
conda create -y -n py37_rdkit_beta python=3.7
conda activate py37_rdkit_beta
conda install -y -c rdkit/label/beta rdkit
```

If you have other Python packages that you'd like to work with, go ahead and install them into the environment now.

Next we copy the RDKit binary components from that environment into our local clone of the RDKit repo:

```
cd $CONDA_PREFIX/lib/python3.7/site-packages/rdkit
rsync -a -m --include '*/' --include='*.so' --include='inchi.py' --exclude='*' . $RDBASE/rdkit
```

NOTE: that rsync command should be one long line.

Finally we set our PYTHONPATH and then test that everything is working by importing the RDKit's Chem module:

```
export PYTHONPATH="$RDBASE"
cd $RDBASE/rdkit
python -c 'from rdkit import Chem;print(Chem.__file__)'
```

That last command should not generate errors and should show you a filename that is in your local github clone. As an example, I started the first step of this process in my /scratch/rdkit_devel directory, so I see:

```
/scratch/rdkit_devel/rdkit/rdkit/Chem/__init__.py
```

# Running the tests
If you're planning on making an RDKit contribution, it's important to know how to run the Python tests to make sure that your changes work and don't break anything else. For historic reasons the RDKit uses a self-written framework for running tests, but it's easy enough to use. You need to run the script  $RDBASE/rdkit/TestRunner.py and point it to the test_list.py file containing the tests to be run. For example, if you want to run all the tests in the directory $RDBASE/rdkit/Chem (this corresponds to the python module rdkit.Chem), you would do:

```
cd $RDBASE/rdkit/Chem
python $RDBASE/rdkit/TestRunner.py test_list.py
```

That will take a while and generate a lot of output, including things that look like exceptions and errors, but should finish with something like:

```
Script: test_list.py.  Passed 40 tests in 69.70 seconds
```

# Finishing up
You're set. The one thing to remember is that whenever you want to use this environment in a new terminal window or shell, you need to activate the py37_rdkit_beta conda environment (don't delete it!), set RDBASE, and set your PYTHONPATH:

```
conda activate py37_rdkit_beta
cd your_local_rdkit_clone  # <- replace this with the real name of the directory
export RDBASE=`pwd`
export PYTHONPATH="$RDBASE"
```

# The recipe
Here's the complete recipe:

'''
git clone https://github.com/rdkit/rdkit.git
cd rdkit
export RDBASE=`pwd`
conda create -y -n py37_rdkit_beta python=3.7
conda activate py37_rdkit_beta
conda install -y -c rdkit/label/beta rdkit
cd $CONDA_PREFIX/lib/python3.7/site-packages/rdkit
rsync -a -m --include '*/' --include='*.so' --include='inchi.py' --exclude='*' . $RDBASE/rdkit
export PYTHONPATH="$RDBASE"
cd $RDBASE/rdkit
python -c 'from rdkit import Chem;print(Chem.__file__)'
'''