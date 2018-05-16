---
title: "Docker commands every dev should know"
layout: post
tag:
- Docker
category: [Tutorials]
comments: true
---

Docker is a really awesome containerization platform. It dutifully simplifies LXC (Linux Containers) and enables developers to develop faster. That said, at times Docker can be a tad confusing and things can get out of hand quickly if you're not up to speed on your docker commands. These are all commands that I keep up to date on my GitHub gists page for docker, so I use them regularly as well.

<!--more-->

## Logging into a container

It's often really helpful to be able to log in to a container. I'll often preload my images destined for a lower stage environment with some troubleshooting tools so I can have visibility into how incoming requests are being handled, etc...

`docker exec -it my-app-container bash`

You can replace `bash` with `sh` if you're on something like Alpine.

## List containers by status and delete them

If you've been working on a build machine like Jenkins and haven't done any clean up in a while you'll probably notice a lot of what I'm about to cover.

`docker ps -aq -f status=exited`

You'd be surprised how many folks aren't used to doing clean up at the end of their Jenkinsfile. No worries though, we can delete all these too!

`docker rm $(docker ps -aq -f status=exited)`

The `$()` causes the command inside it to be executed and everything on the outside to be run iteratively over the results. Docker was fashioned in a way to make things like this possible.

## But volumes!

Volumes are not deleted alongside containers. You have to delete these separately. Once you delete a container and that volume is no longer associated to any container the volume is known to be in a **dangling** state. We can remove all these fairly easily:

`docker volume rm $(docker volume ls -qf dangling=true)`

## Don't forget images

Images are the last to remain and just easy to clean up. Images will frequently go into a dangling state as well when they're no longer associated with a container.

`docker rmi $(docker images --filter "dangling=true" -q --no-trunc)`

As of Docker 1.13 you can also simplify all of the above using `docker network prune` and `docker system prune`. That said, if you're using a version prior to 1.13 and you encounter problems with images in use that aren't really in use, simple tack -f or --force onto the rm or rmi command.

I hope this has been educational for everyone!
