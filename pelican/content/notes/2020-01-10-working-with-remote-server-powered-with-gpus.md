---
layout: post
visible: False
date: 2020-01-10 20:00:00 +0800
tags: ML
title: Working with remote server powered with GPUs
---
## Package management


## Conda related Commands  
As a non-root user, I don't have root access to the whole server. Installations are limited to my sub-folder only. To be able to work in an environment of more flexibility, a Conda environment needs to be created.

First, you would like to check the existing environments with the following command to list all of them.  
`conda info --envs`  

You can create an environment of name "env_name" with  
`conda create --name env_name`  

Activate and de-activate to dive in the environment  
`conda activate(deactivate) env_name`  
Once your are in the environment, the environment name will show before the $ sign.  
