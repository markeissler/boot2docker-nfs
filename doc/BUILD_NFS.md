# How to build boot2docker-nfs.iso locally

The __boot2docker-nfs.iso__ is built on top of the standard __boot2docker.iso__
which is itself built with Docker, via a Dockerfile.

For more information on how the standard __boot2docker.iso__ is built, see [How to build](doc/BUILD.md).

## Building the ISO

The process of building the __boot2docker-nfs.iso__ is just as simple as any
other build and because the process leverages the already built standard
ISO build time is relatively short.

From the repo's top-level directory issue the
following commands:

```sh
prompt> docker build -f Dockerfile.nfs -t boot2docker-nfs-img .
prompt> docker run --rm boot2docker-nfs-img > boot2docker-nfs.iso
```

## Testing with docker-machine

To test the produced __boot2docker-nfs.iso__ locally with `docker-machine` just
specify an alternative `--DRIVER-boot2docker-url`. For instance, with [VMWare
Fusion](http://www.vmware.com/products/fusion.html) the command line would be:


### Create a machine

```sh
prompt> docker-machine create --driver vmwarefusion \
    --vmwarefusion-boot2docker-url=./boot2docker-nfs.iso nfs-iso-test
```

Import the machine's environment:

```sh
prompt> eval $(docker-machine env nfs-iso-test)
```

### Create a docker volume with an NFS source

Prepare your NFS server accordingly. In the following example, our target NFS
volume has a path of `/opt/share/test`, and our NFS server is located on the
local private network at address `10.0.1.2`.

The following file exists on the shared volume:

```sh
prompt> ls -la /opt/share/test
total 0
drwxr-xr-x  3 root  wheel  102 Jun 10 12:43 .
drwxr-xr-x  4 root  wheel  136 Jun 10 12:42 ..
-rw-r--r--  1 root  wheel    0 Jun 10 12:43 myfile.txt
```

Create a docker named volume:

```sh
prompt> docker volume create \
    --driver local \
    --opt type=nfs \
    --opt o=addr=10.0.1.2,rw \
    --opt device=:/opt/share/test \
    --name foo
foo
```

### Attach the docker volume to a container

Launch an instance of [alpine linux](https://alpinelinux.org/) and attach the
named volume:

```sh
prompt> docker run -it -v foo:/mnt/test alpine:latest /bin/sh
/ # ls -la /mnt/test/
total 8
drwxr-xr-x    3 root     root           102 Jun 10 19:43 .
drwxr-xr-x    3 root     root          4096 Jun 10 19:49 ..
-rw-r--r--    1 root     root             0 Jun 10 19:43 myfile.txt
/ #
```

Remember, the NFS volume is attached the machine, not the container. Traditional
docker data volume sharing is used to attach the volume to the container.

### Cleanup after your test

You will need to cleanup after running the above test steps. Once you've exited
the running container, clean it up as usual:

```sh
prompt> docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Mounts}}"
CONTAINER ID        IMAGE               MOUNTS
fcdf5e0d6952        alpine:latest       foo
6bec897bd7ca        3da34c969384        f4e6549a5b04...,aee3ef8385af...
...
```

The above command line will output all running and non-running container, along
with the names of any volumes mounted on the containers. If you have a lot of
containers starting and stopping then this let's you verify which container you
are about to remove (based on the volume name).

Remove the container:

```sh
prompt> docker rm fcdf5e0d6952
```

A shorter command that finds the ID of the last container started:

```sh
prompt> docker rm $(docker ps -l -q)
```

Obviously, only use the above command if you are confident the last container is
the one you wish to target for removal.

The docker volume can be removed with:

```sh
prompt> docker volume rm foo
```

## Making your own customised boot2docker-nfs ISO

For the moment, you must build __boot2docker-nfs.iso__ from scratch. If you
you need/want to customize the build you should base your work on the existing
`Dockerfile.nfs` file.
