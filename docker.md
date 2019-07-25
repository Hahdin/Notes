# Docker commands

## Installation

```bash
sudo yum update -y
sudo install docker
sudo service docker start
# add the ec2-user to the docker group
sudo usermod -a -G docker ec2-user
# restart/re log to pickup the new permissions

#test if you have permission, should not need sudo now
docker info 
```

## Create a docker file

```bash

touch docker
```

Edit the file, simply test below for amz linux AMI
```
FROM amazonlinux:2018.03
MAINTAINER You <you@you.com>

CMD ["echo", "hello from CMD"]

```

```bash
# create the image with a tag (0.1), add a space and . to indicate current dir
docker build -t testimage:0.1 .
# check if the image was created
docker images --filter reference=testimage

# and you should see something like:
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
testimage           0.1                 57f0f3a57abd        About a minute ago   167MB
```

Run the image to test it

```bash 
docker run testimage:0.1
# .. should output

hello from CMD

```

# clean up

```bash 
# remove the image now, use -f to force it
 docker rmi -f 57f0f3a57abd
```


