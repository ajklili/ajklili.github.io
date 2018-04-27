---
layout: post
title:  "CUDA Setup Example"
date:   2018-04-13
categories:
tags:
excerpt: My script to setup CUDA on gcloud instances
mathjax: false
---

* content
{:toc}

## Script

```bash
#!/bin/bash
echo "Checking for CUDA and installing."


# Check for CUDA and try to install.
curl -O http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1704/x86_64/cuda-repo-ubuntu1704_9.1.85-1_amd64.deb
sudo apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1704/x86_64/7fa2af80.pub
sudo dpkg -i ./cuda-repo-ubuntu1704_9.1.85-1_amd64.deb
sudo apt-get update
sudo apt-get install cuda-9.1 -y
rm -rf ./cuda-repo-ubuntu1704_9.1.85-1_amd64.deb


# install cuDNN
# https://gist.github.com/sayef/8fc3791149876edc449052c3d6823d40
curl -fsSL http://developer.download.nvidia.com/compute/redist/cudnn/v7.1.3/cudnn-9.1-linux-x64-v7.1.tgz -O
tar -xzvf cudnn-9.1-linux-x64-v7.1.tgz
sudo cp -P cuda/include/cudnn.h /usr/local/cuda-9.1/include
sudo cp -P cuda/lib64/libcudnn* /usr/local/cuda-9.1/lib64/
sudo chmod a+r /usr/local/cuda-9.1/lib64/libcudnn*


# set environment variables
export PATH=/usr/local/cuda-9.1/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.1/lib64/${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
nvidia-smi


# install python
sudo apt-get -y install libcupti-dev
sudo apt-get -y install python3-pip python3-dev python-pip
sudo pip install gpustat
gpustat
```

