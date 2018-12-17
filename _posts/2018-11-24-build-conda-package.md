---
layout: post
title:  "Building a conda package and uploading it to Anaconda Cloud"
author: qiusheng
categories: [ geospatial, tutorial, programming, linux, python ]
image: "https://i.imgur.com/oOxf9YM.png"
---

How to build a conda package from an existing PyPI package and upload it to Anaconda Cloud?  

I have developed several Python packages (e.g., [lidar](https://pypi.org/project/lidar/), [whitebox](https://pypi.org/project/whitebox/), [pygis](https://pypi.org/project/pygis/)) and made them available on the [Python Package Index (PyPI)](https://pypi.org/). Python packages available on PyPI can be easily installed using the following pip command: 

```
pip install package-name
````

[Conda](https://conda.io/docs/) is another open source package management system and environment management system. Python packages available on [Anaconda Cloud](https://anaconda.org) or [conda-forge](https://conda-forge.org/) can be easily installed using the conda command: 

```
conda install -c channel-name package-name
```

I have been thinking about deploying my existing Python packages to Anaconda Cloud for some time. Today I followed the [official conda tutorial](https://conda.io/docs/user-guide/tutorials/build-pkgs-skeleton.html) to build the [whitebox](https://anaconda.org/giswqs/whitebox) conda package and upload it the Anaconda cloud. The official conda tutorial is helpful, but it becomes very time-consuming when you are trying to build a package for multiple platforms (e.g., Linux, Windows, macOS) with different Python versions. Therefore, I created a bash script to automate the process. Here are the two steps:

**Step 1: Install the prerequisites**

Before you start the tutorials, you should already have installed [Miniconda](https://conda.io/docs/user-guide/install/index.html) or [Anaconda](https://docs.continuum.io/anaconda/install).
You also need to install conda build and anaconda-client using the following commands:

```
sudo conda install conda-build
conda install anaconda-client
anaconda login
```

**Step 2: Run the bash script**

There are two parameters you need to change before running the bash script: `pkg` (i.e., PyPI package name) and `array` (i.e., Python versions). **Note that all dependencies for your Python package must already exist on Anaconda Cloud or conda-forge**. Otherwise, conda build will fail.   


Download the following bash script from [here](https://gist.github.com/giswqs/4eb62fb08658c8a200c4e18bb5e6270c).
```
#!/bin/bash

# change the package name to the existing PyPi package you would like to build
pkg='whitebox'
# adjust the Python versions you would like to build
array=( 3.5 3.6 3.7 )

echo "Building conda package ..."
cd ~
conda skeleton pypi $pkg
cd $pkg
wget https://conda.io/docs/_downloads/build1.sh
wget https://conda.io/docs/_downloads/bld.bat
cd ~

# building conda packages
for i in "${array[@]}"
do
	conda-build --python $i $pkg
done

# convert package to other platforms
cd ~
platforms=( osx-64 linux-32 linux-64 win-32 win-64 )
find $HOME/conda-bld/linux-64/ -name *.tar.bz2 | while read file
do
    echo $file
    #conda convert --platform all $file  -o $HOME/conda-bld/
    for platform in "${platforms[@]}"
    do
       conda convert --platform $platform $file  -o $HOME/conda-bld/
    done    
done

# upload packages to conda
find $HOME/conda-bld/ -name *.tar.bz2 | while read file
do
    echo $file
    anaconda upload $file
done

echo "Building conda package done!"
```

**Step 3: Install the conda package**

Once the conda package is successfully built and uploaded to Anaconda Cloud, you can then install the package using the `conda install` command, such as 

```
conda install -c giswqs whitebox
```

Checkout the **whitebox** package on [Anaconda Cloud](https://anaconda.org/giswqs/whitebox).

![](https://i.imgur.com/oOxf9YM.png)