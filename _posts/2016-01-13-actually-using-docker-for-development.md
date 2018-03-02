---
id: 518
title: (Actually) using Docker for Development
date: 2016-01-13T10:17:43+00:00
author: Robert
layout: post
guid: http://wiredtron.com/?p=518
permalink: /2016/01/13/actually-using-docker-for-development/
categories:
  - Docker
---
A lot of the material I have read involving using docker with rails for development has left me cursing at the screen unnecessarily. Hopefully this post will not give you the same reaction. A lot of the problems I encountered with the posts I founded ended up boiling down to one of two things.
  
1) They were using docker compose or some other additional functionality across multiple machines that I didn&#8217;t need.
  
2) They didn&#8217;t address the issue of actually editing files on your local system and keeping them in sync with your remote docker box. (Hint: using volumes)

First up, I have created a working prototype of what I am talking about and put it on GitHub <a href="https://github.com/rbclark/docker-rails-development-demo" target="_blank">here</a>. The README should be sufficient for how to use it, however in order to better explain the reasoning behind it, I figured a blog post would be useful.

The whole purpose of this was having the ability to work on your local machine and have all of your files synced to the thing actually hosting the server. Before Docker this would&#8217;ve been the equivalent editing files locally and having them automatically FTP&#8217;d back to your development server, or something along those lines. This is desirable since your local OS isn&#8217;t always exactly the same as the remote one and some rails gems and things work better on a linux based OS. Also it seems desirable to me to perform your development in a very similar environment to the one which you will be deploying to.

Now how does this actually work in practice with Docker? Well we end up using Docker a bit differently than most people do in order to make this work. Normally when using Docker you just pull down an image from the Docker Hub and use docker run, however in this case we need to actually build our own very simple image. Depending on how often your gems change this could be a bit of a sore spot, however the good news is you can rebuild the bundle inside of the docker container whenever you want and then only rebuild the complete image once you have finished tweaking your gems.

Once you have built your image containing your bundle, you then can run it and take advantage of Docker volumes. Docker volumes allow you to keep directories in your Docker container in sync with the host machine. This means you can edit your code on your local system and have it show up on your Docker system without having to worry about how it got there.

The one thing I have not mentioned yet is databases and specifically how they fit into this method of development. Fortunately if you use any flat file type of database for your project where the database is stored in your local folder on the Docker box (as is the case with sqlite) you are in luck. The other upside here is that sine sqlite is the default, theres a good chance most projects are already using this for development. The downside is if any other database is being used for development that is not a flat file database, then this adds some complexity since the database will need to be built into the image you are using.

I believe this is a very useful method for using Docker in development, mainly useful for people wanting to do rails development on a Windows machine, since I have found it to be a lot easier to install Docker on Windows than to install ruby + rails + get gems working on Windows.