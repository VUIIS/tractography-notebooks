tractography-notebooks
======================

Notebooks for performing whole-brain tractography

I've used this group of notebooks to implement cortical tractography. There is little to no useful information about how to actually do this so I hope these notebooks are helpful to the community.

Under no circumstance should this be taken as "correct", rather it's my best attempt at a processing stream for cortical tractography. However, I don't think it's necessarily wrong either. Feel free to pick and choose what you do or don't like.

The steps implemented in these notebooks are written about [here](http://sburns.org/2014/05/03/cortical-tractography-recipe.html).

## Setup

You'll need the full Python scientific stack (numpy, pandas, mpl, etc). Consider [Anaconda](https://store.continuum.io/cshop/anaconda/) or it's smaller brother, [conda](http://conda.pydata.org) to install these.

At first, these 5 notebooks were one gargantuan notebook. I quickly realized that because there are discrete steps to this process, I realized I needed discrete notebooks. However, out-of-the-box python cannot import code from other notebooks. If you're running IPython (and you **really** should be), follow [this gist](https://gist.github.com/sburns/e889642c48aa3b19a36f) to setup your interpreter such that you can import notebooks.

## Notebooks

### common.ipynb

### 00_Bedpost.ipynb

### 01_Setup.ipynb

### 02_ProbTrack.ipynb

### 03_SingleSubjectAnalysis.ipynb

### 04_ProcessingTime.ipynb

### 05_GroupBehavioralAnalysis.ipynb