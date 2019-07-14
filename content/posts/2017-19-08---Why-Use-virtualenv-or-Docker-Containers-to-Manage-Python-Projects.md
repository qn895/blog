---
title: Why Use virtualenv or Docker Containers to Manage Python Projects
date: "2019-07-14T22:40:32.169Z"
template: "post"
draft: false
slug: "/posts/using-docker-containers-for-python-projects/"
category: "Random Musings"
tags:
  - "Docker"
  - "Python"
  - "JupyterLab"
description: "Having dedicated Docker containers for your python projects instead of relying on virtual environments and anaconda will save your insanity and significantly speed up share-ability. Here I share some snippets that will speed up your set up."
---

A few years ago, I started working on a few large-scale Python projects for my internship. Some involved legacy scripts which used Python 2.7, and some were projects I started which heavily used Python's interactive scientific stack and Jupyter notebooks. Because every project is different with different dependencies, I opted to use Anaconda's virtualenv for each project so it's easier to maintain.

The main reasons of having [**dedicated isolated environments**](https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/) instead of having packages installed globally are very simple:

1. **It's easier to switch between different versions of Python**. This was especially relevant if you deal with legacy code which has syntax from older versions.

2. **It's easier to maintain dependencies and different package versions**. A very common problem you might run into is if ProjectA uses `genericpkg=1.0.0` while ProjectB uses `genericpkg=2.0.0`. If you install this `genericpkg` globally using pip (which by the way, might become `genericpkg=3.0.0` by the time you run the command years alter), you might run into unexpected bugs and errors when you run either of your project. As such, it's much better to have an isolated environment for each project with a `requirements.txt` of packages and corresponding versions at the point of time you write the code.

I was fairly happy with this set up, until I need to install external drivers (e.g. Oracle's JDBC driver, Spark, Hadoop, AWS, etc) and quickly found out how tedious it was to replicate my setup with my co-workers. Virtualenv is great for isolating and managing Python dependencies, but Docker is better when you need to replicate the whole OS set up that's outside of Python's scope.

Did I mention it's easier to bootstrap projects using Docker? Let me repeat it again: **It's easier to share your Python set up with someone else and have that person get it up an running within a minute**. No longer will they need to install Anaconda, download external drivers, install packages, and a whole list of items, all they need to do is `docker-run`!

Additionally, I found it much easier to deploy my notebooks to my company's internal web server with Docker. Now, this was the dark age before [Google's Colaboratory](https://colab.research.google.com/notebooks) or [Microsoft's Notebooks](https://notebooks.azure.com/), but still, these are proprietary code that could only be run privately within the comapny's network. With Docker, pusing images and hosting them with Kubernetes was a breeze.

Finally, as our projects evolve over time from simple Python programs into Flask microservices, Docker has become the defacto way for our team members. Each microservice is a Docker container/nginx server we can integrate with Git hooks and Jenkins to automatically deploy any new changes.

So overall, while Python's virtualenv provides a nifty way to isolate python dependencies, I recommend using containers like Docker even for simple Python projects because it provides more scalability in the long run :)

If you're interested in checking out how to use Docker containers for your project, I shared a few [useful snippets to get things up and running in this blog post.](/posts/userful-docker-nippets-bootstrap-python-projects/)
