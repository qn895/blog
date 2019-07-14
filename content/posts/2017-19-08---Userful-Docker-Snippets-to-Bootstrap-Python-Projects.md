---
title: Userful Docker Snippets to Bootstrap Python Jupyter Projects
date: "2019-07-14T22:40:32.169Z"
template: "post"
draft: false
slug: "/posts/userful-docker-nippets-bootstrap-python-projects/"
category: "Code Snippets"
tags:
  - "Docker"
  - "Python"
  - "Jupyter"
description: "Having dedicated Docker containers for your python projects instead of relying on virtual environments and anaconda will save your insanity and significantly speed up share-ability. Here I share some snippets that will speed up your set up."
---

Here's a few useful scripts for I use to bootstrap a new project with a dedicated JupyterLab instance using Docker. If you're on a Mac, copy these files and call it in your terminal.

### bootstrap.sh

I use this script to create a new folder with a `Dockerfile`, a `build` script, and a `start` script.

```Shell
if [ -z ${1} ]; then NOTEBOOK_NAME="my-project"; else NOTEBOOK_NAME=$1; fi

mkdir $NOTEBOOK_NAME
cd $NOTEBOOK_NAME
# export NOTEBOOK_NAME=${PWD##*/} # use the current folder's name as the project name
echo "Creating new project:" $NOTEBOOK_NAME
echo '# Start from a core stack version
FROM jupyter/datascience-notebook:9f9e5ca8fe5a
# Install in the default python3 environment
' > Dockerfile
echo 'docker build --rm -t jupyter/'${NOTEBOOK_NAME}' .' > build.sh
echo 'docker run -it -v /$(pwd):/home/jovyan -p 8888:8888 jupyter/'$NOTEBOOK_NAME':latest start.sh jupyter lab' > run.sh
source build.sh
source run.sh
echo "Finished bootstrap new folder:" $NOTEBOOK_NAME
```

To run it in terminal you can do `sh boostrap.sh my-custom-name` where `my-custom-name` is an optional argument to name your project. After running this the first time, you only have to run `sh run.sh` afterwards.

### Dockerfile

In the bootstrap script above, I'm not installing any extra packages. If you wish to start the image with some packages or `requirements.txt` instead, you can refer to the [recipes contributed by the community here](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/recipes.html#).

```dockerfile
# https://jupyter-docker-stacks.readthedocs.io/en/latest/using/recipes.html#using-pip-install-or-conda-install-in-a-child-docker-image
# Start from a core stack version
FROM jupyter/datascience-notebook:9f9e5ca8fe5a
# Install in the default python3 environment
RUN pip install 'opencv-python'
```

### build.sh

This will build the Docker image from the docker file.

```shell
docker build --rm -t jupyter/my-project-name .
```

### run.sh

This will start the docker container from the built image. You need to run build first before running this.

Note: the `-v /$(pwd):/home/jovyan` is important here. It mounts the current working folder to the Docker environment, so any changes you make to the folder will be updated in the docker container and vice versa. If you don't have this, you won't be able to call code or assets from your computer, and changes to your files will be lost when you kill the session.

Here I use `jupyter lab` out of personal preference, but you can replace it with `jupyter notebook` instead.

```shell
docker run -it -v /$(pwd):/home/jovyan -p 8888:8888 jupyter/my-project-name:latest start.sh jupyter lab
```
