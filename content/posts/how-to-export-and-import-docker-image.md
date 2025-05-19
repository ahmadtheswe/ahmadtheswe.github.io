+++
date = '2023-11-25T10:57:49+07:00'
draft = false
title = 'How to Export & Import Docker Image'
author = 'ahmad'
categories = ['docker', 'docker-image']
tags = ['docker', 'docker-image']
featuredImage = '/images/how-to-export-and-import-docker-image/featured.png'
+++

We can always pull docker images from docker registry. But, sometimes we need to share our images manually. In this post, I will share you how to import and export docker images.

## Export Docker Image
Let say we have these two docker images in my local docker
```bash
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
mysql         5.7       547b3c3c15a9   4 weeks ago     501MB
postgres      12        d480c196f9b0   3 months ago    405MB
```
We want to export postgres docker image. To do that, we can do that using `docker export` command like this :
```bash
$ docker export postgres > postgres.tar
```

Here are the explanation for this command. In general the command is like this :
```bash
$ docker export <image-name> > <output-name>.tar
```
* change `<image-name>` with the image that you want to export
* change `<output-name>` in to the output name you intended. The output file will be on `.tar` format.

## Import Docker Image
Let say you want to import / load the exported docker image file (the `.tar` file). You can use `docker load` command like this :
```bash
$ docker load -i postgres.tar
```
In general, the command is like this :
```bash
$ docker load -i <tar-file>.tar
```
If the import / load is successful, you will see the image if you run `docker images` command.


That's all, very simple :)

<hr>

External sources :
* [docker export](https://docs.docker.com/engine/reference/commandline/export/)
* [docker load](https://docs.docker.com/engine/reference/commandline/load/)
