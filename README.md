tractography-notebooks
======================

I've used this group of notebooks to implement cortical tractography. There is little to no useful information about how to **actually perform** this kind of analysis so I hope the community finds these notebooks helpful.

Under no circumstance should this be taken as "correct", rather it's my best attempt at a processing stream for cortical tractography. I don't think it's necessarily wrong either. Feel free to pick and choose what you do or don't like.

A less-techincal, more verbose description of the steps implemented can be found [here](http://sburns.org/2014/05/03/cortical-tractography-recipe.html).

## Setup

You'll need the full Python scientific stack (numpy, pandas, mpl, etc). Consider [Anaconda](https://store.continuum.io/cshop/anaconda/) or it's smaller brother, [conda](http://conda.pydata.org) to install these.

Most everything in here depends on [FSL5](http://fsl.fmrib.ox.ac.uk/fsl/fslwiki/).

At first, these 5 notebooks were one gargantuan notebook. I quickly realized that because there are discrete steps to this process, I needed discrete notebooks. But, there's a lot of interlocking pieces (paths to data, directories, scripts, etc) and not repeat yourself when you tear apart the notebooks, we need to somehow reference code in other notebooks. Out-of-the-box python cannot import code from other notebooks :(.  However, if you're running IPython (and you **really** should be), follow [this gist](https://gist.github.com/sburns/e889642c48aa3b19a36f) to setup your interpreter so that you can import notebooks.

## Required Data

Each subject needs a diffusion-weighted image and a high-res T1 with the cortical surface reconstructed with [Freesufer](http://freesurfer.net). Obviously you'll need the gradient values & directions as well.

I've run this on a 60-gradient HARDI sequence. With good SNR I'm sure you could get by with a standard 32-direction Jones sequence.

## Notebooks

### common.ipynb

[In NBViewer](http://nbviewer.ipython.org/github/VUIIS/tractography-notebooks/blob/master/common.ipynb)

Common functions for determining directories, script paths, etc. To get this going on your data, you **must** change these functions so they work for however your data is organized.

Theoretically, after you alter this notebook, the rest of the notebooks will just work &reg; (with some caveats).

### 00_Bedpost.ipynb

[In NBViewer](http://nbviewer.ipython.org/github/VUIIS/tractography-notebooks/blob/master/00_Bedpost.ipynb)

Bedpostx, when not accelerated, takes between 20-30 minutes a slice and for a normal (50 slice?) dataset, that's about 17 hours. **But**, each slice is processed totally independently, so there's a linear speed-up to be had if you parallelize across cores of your machine. This notebook implements such a version of bedpost. Yes, it's crazy. Yes, it works and makes the time needed for bedpostx to complete about 90 minutes, give or take.

More sane people should just use the GPU variant of bedpostx. You've been warned.

FYI before bedpostx, we also run `eddy_correct` on the data. Again, if you have a better idea, make it happen.

### 01_Setup.ipynb

[In NBViewer](http://nbviewer.ipython.org/github/VUIIS/tractography-notebooks/blob/master/01_Setup.ipynb)

This implements a script that does the following:

1. convert the structural & `orig.mgz` to nii.gz, moving them into an `anat` directory.
1. Binarize the `aparcaseg.mgz` image into `ventricles.nii.gz`, `lh.wm.nii.gz` & `rh.wm.nii.gz` images. The paths to white-matter images are cat'd into `waypoints.txt`
1. Runs dtifit real fast to get a FA image for registration. If you're making FA images elsewhere, feel free to get rid of this.
1. Register the raw T1 to the re-sliced `fs.nii.gz` with `tkregister2`. **Shouldn't** fail but you never know. Invert to produce `fs2str.mat`
1. Register FA to structural (not FS structural though) with a 6-dof `flirt` to produce `fa2str.mat`. Invert to produce `str2fa.mat`. Knock yourself out if you want to use `fnirt`.
1. Concatenate & invert to produce `fa2fs.mat`. Apply to FA & `slicer` the overlay. You **must** take a look at this picture and verify the registration. Murphy's law says if you don't, you will get crap probtracking results. You've been warned.
1. Then for each hemisphere & label we convert the labels to nii.gz. I've found using surface gifti files doubles `probtrackx2` memory usage to roughly 23GB and did not find significant differences between the results. `cat` all these volumes into your `seeds.txt` file.


### 02_ProbTrack.ipynb

[In NBViewer](http://nbviewer.ipython.org/github/VUIIS/tractography-notebooks/blob/master/02_ProbTrack.ipynb)

This writes out all 148 PBS scripts for tractography. Honestly, you can't do this analysis without a sizable compute cluster at your disposal as total compute time is around a month **per subject**.

If you use a different interface to your compute cluster than `qsub`, you'll need to change this notebook.

### 03_SingleSubjectAnalysis.ipynb

[In NBViewer](http://nbviewer.ipython.org/github/VUIIS/tractography-notebooks/blob/master/03_SingleSubjectAnalysis.ipynb)

The `single_plot` function in this notebook creates a [plot like this](http://sburns.org/assets/img/adj.png). It's build the 148x148 matrix by computing what others call "streamline-based connectivity index". Collapse the `matrix_seeds_to_all_targets.nii.gz` from SeedVoxelsXTargets to 1XTargets by suming & divide by total (accepted) streamlines from the seed. I don't normalize by (# seed voxels + # target voxels). If you want to, go for it.

### 04_ProcessingTime.ipynb

[In NBViewer](http://nbviewer.ipython.org/github/VUIIS/tractography-notebooks/blob/master/04_ProcessingTime.ipynb)
Simple little notebook to plot processing time per run. Was trying to get a better idea about time required per size of seed. Theoretically you could build more accurate walltimes into your PBS scripts by taking into account the size of the seed for that run.

### 05_GroupBehavioralAnalysis.ipynb

[In NBViewer](https://github.com/VUIIS/tractography-notebooks/blob/master/05_GroupBehavioralAnalysis.ipynb)
The most-dumb and quickest analysis of this data is to stack the 148x148 matrices into a 148x148xNsubject matrix & covary the seed of interest-to-target of interest connectivity measure with some kind of out-of-magnet behavior.

Fancier methods might include turning the adjacency matrix into a graph and walking from one ROI to another, possibly comparing this "walking difficulty" between groups? Hopefully Olaf Sporns will review your paper :)
