# Welcome To Kara Org CTF Writeup

> this is a writeup for ‘Welcome To Kara Org’ challenge from yogasha Christmas CTF

The challenge category is OSINT, Forensics, the challenge talks about the big threat to Konoha, Buroto and there are in the docker image provided

> Welcome To Kara Organization! It’s the Christmas so Konoha will let its guards down and we can take down the big threat Boruto! For that we need to prepare our forbidden jutsus. In Kara organization we are well secure, we have hidden some clues in this **dock**u**er**u image shisuiyogo/christmas , can you retrieve them as your first mission ?

The first thing I did was pull the image and run it
```bash
docker run --rm -it shisuiyogo/christmas
```
I found that the image was made for ARM arch while mine is x86_64, I found a [writeup](https://matchboxdorry.gitbooks.io/matchboxblog/content/blogs/build_and_run_arm_images.html) for this just had to install **qemu-user-static** package, I did some patterns search through the image for any applications or something interesting but found nothing, I don’t have experience with docker or digital forensics so started doing some research about it, the first interesting found was [**dive**](https://github.com/wagoodman/dive) tool that is used to explore layers in docker

> A tool for exploring a docker image, layer contents, and discovering ways to shrink the size of your Docker/OCI image.

layers? the layers are the states or changes that happen to the base image during running the DockerFile, and each layer is an image itself. They have auto-generated IDs though. [docker image layers](https://vsupalov.com/docker-image-layers/)
![[Pasted image 20221228135050.png]]
There are three layers in this image, what is interesting is the secret_notes.txt file is copied to the container and deleted in the next layer, we could come up with the same finding if we went to [docker hub profile](https://hub.docker.com/layers/shisuiyogo/christmas/latest/images/sha256-3111e65a2097ecfe94d9326b8a101c7bbb86746a0872fd793335cba8dac1b31f) and check the layers in the image details.

as I didn’t have enough experience I thought I could edit the layers and remove the last one, after a bit of researching I came out with this conclusion:

-   we can edit an existing image that we can’t rebuild by adding more layers that modify the resulting image, but we can’t do anything about deleted and overwritten files
-   we can add layers using [**docker commit**](https://docs.docker.com/engine/reference/commandline/commit/) to modify images

The idea of editing layers won’t work, I had to come up with another solution because other players were able to solve the challenge in a minute, so there should be something I missed, which was searching through the pulled image files!
I did a quick google search for where docker images are stored and the first result was **/var/lib/docker/** I did a quick search using find
```bash
find /var/lib/docker/ -name secret_note.txt
```

and I found the file exists in two folders
![[Pasted image 20221228135416.png]]
The first one is a character special file, to understand the character special file read, to know about the special character files check this [answer](https://unix.stackexchange.com/a/60147)
![[Pasted image 20221228135438.png]]
by reading the second file, we got the Flag!
![[Pasted image 20221228135448.png]]

The challenge was nice and simple, it only required a bit of experience with docker or having the time to do some research!

