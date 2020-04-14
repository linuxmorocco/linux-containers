# linux-containers
Summary of our first workshop about linux containers

# Slides
1. [Linux Namespaces](https://docs.google.com/presentation/d/1WglBqb8lPGdp8q2d3c2b1MEsP5gwIVUhEaR8_3oEA8E/edit#slide=id.p)
2. [SELinux](https://docs.google.com/presentation/d/1AalhlPGuTHx1q7tjL_VI9eDVAKxGONHeY428Gfls7u4/edit#slide=id.g72af2cb99a_0_71)
3. [Better way to learn containers](https://docs.google.com/presentation/d/1tj_N65-Rx8ZNc4YVkhQykkrtDS7aal0YhMUdJBb0QDo/edit#slide=id.g726da336b0_0_7)

# Workshop (Demos)
## Linux container technology
### Namespaces
- **unshare**: run a program with namespace disassociated from it's parent 
```bash
# run bash in a separate pid namespace 
sudo unshare --fork --pid --mount-proc bash
ps -aux # this will show 2 processes (bash[pid=1] and ps[pid=2])
# run bash in a separate user namespace
sudo unshare --map-root-user --user bash
whoami #this will show root
```
- **nsenter**: run a program with namespaces of another
```bash
docker ps --ns # get namespaces info about contaniners including the pid.
sudo nsenter --target <pid-of-container> --mount --user --uts --ipc --net --pid /bin/sh
ps -aux # it will list the processes run inside the container (inside the pid namespace of the container)
```

### Cgroups
- **Cgroups**: a linux technology used to limit process ressources, cgroups informations are available under `/sys/fs/cgroup/`
```bash
# limit memory to 250M and cpus number to 1
docker run -m 250m --cpus=1 <container-image>  
# disable swap 
docker run -m 250m --memory-swap=250 <container-image>  
```
  - When the container surpass the memory limit the kernel throws a  OOME (Out Of Memory Exception) and starts killing processes starting (most of the time) with the containers (0).
  - If `--memory-swap` is set to the same value of `-m or --memory`. swap is disabled. (0)

### Capabilities
```bash
# get the pid of the container
docker ps --ns
# get and decode Capabilities used by a container. 
cat /proc/<pid-container>/status | grep ^Cap | cut -d: -f2 | xargs -I{} -n1 capsh --decode={}
```
## The anatomy of containers and containers images. 
### Docker images anatomy: 
- Each image is composed from multiple layers (tar files) each with a json file containing metadata. + a manifest containing metadata about the entire image.
- Each tar file contains a file system with files added in the equivalent Docker Command (from the Dockerfile)  
```bash
docker pull busybox # download the image busubox, you can pull whatever image you want
docker save busybox > busybox.tar # save the image into a tar file
tar -xvf busybox.tar -C busybox # extract the tar file
tree # or ls to explore the files 
```
![container image](https://media.discordapp.net/attachments/691698282338058253/696104132192370790/2020-04-04-220706_1600x900_scrot.png?width=1040&height=585)

### Container anatomy: 
- Containers are stored in the file system `/var/lib/containers/storage/overlay-containers` for podman `/var/lib/docker` for docker.
- Container create a rw layer on top of the image layers containing the data written by the container. 
```bash
docker commit container-id # write the data by the rw layer of the container to a new image
docker export container-id > image.tar # export the container to a tar file
import image.tar image-name # import the tar to a image and store it in a repository 
```
## Container tools
### The container lifecycle
pull -> expend -> mount -> create a Spec file (config.json) -> run

### Container tools
- **Container engine** (containerd (docker), podman, Cri-O ...): is responsible of  pull -> expend -> mount -> create a Spec file (config.json)
- **Container runtime** (runc, crun, katacontainers ...): reads the spec file and instructs the kernel.
linux kernel: responsible for running and killing the container.
- **linux kernel**: responsible for running and killing the container.
```bash
watch -n 0.1 -e ps aux | grep "runc|crun" # watch the container runtime in a separate terminal
docker run -it <image> bash # you'll notice runc appearing and disapearing briefly in the watch
```
[[0]](https://docs.docker.com/config/containers/resource_constraints/)
# Ressources
## Linux container technologies
[Security-Enhanced Linux for mere mortals (Red Hat Summit) - Youtube](https://www.youtube.com/watch?v=_WOKRaM-HI4)

[Using SELinux with container runtimes (DevConf) - YouTube](https://www.youtube.com/watch?v=FOny29a31ls)

[How Docker Works - Intro to Namespaces (LiveOverflow) - Youtube](

[Containers unplugged: Linux namespaces (NDC Conferences / Michael Kerrisk) - Youtube](https://www.youtube.com/watch?v=0kJPa-1FuoI)

[Cgroups, namespaces, and beyond: what are containers made from? -Youtube](https://www.youtube.com/watch?v=sK5i-N34im8)

## Linux Containers
[Interactive Courses about containers - Katacoda](katacoda.com/)
