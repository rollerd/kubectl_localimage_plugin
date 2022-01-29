## kubectl-localimage plugin

Sometimes when troubleshooting or playing around with a remote Kuberenetes cluster, you would like to be able to run a container image that you have in your local docker registry on the cluster but:

- you don't want to go through the hassle of pushing it to a remote registry before Kubernetes can pull it

- you may have sensitive data in the container (not a good idea to begin with) and don't want to upload it to a repo

localimage plugin is a crude way of getting your local image onto the remote Kuberenetes node and running a pod with that image

#### Usage

localimage takes three arguments: 

- local image name:tag (myimage:1.0.0)

- name of pod that will be running the image in the cluster (eg mypod)

- commands to pass to the pod on startup (the part after `--` when using kubectl run. eg '/bin/sleep "6000"')

Example:

```
$ docker images
REPOSITORY                                                                TAG                                        IMAGE ID       CREATED         SIZE
imagetest                                                                 1.0.0                                      81624a2072ba   2 hours ago     7.88MB

$ kubectl localimage localimage:1.0.0 mynewpod /bin/sleep "6000"
```

#### What is the plugin doing

The plugin is just a simple bash script that does the following:

- Creates a pod that has a docker init container running on it which mounts the host nodes docker.sock file. It also runs an sh script which greps for the local image name:tag on the Kube host node and exits once it is found.

- Saves the local docker image to a tar file

- Copies the image tar file to the init container in the pod

- The init container does a 'docker load' on the image file, putting it into the Kube nodes local docker registry

- The init container runs the following in a loop until it exits successfully (indicating the local image is now on the Kube node):

```
until docker images | grep <local_image_name>:\.*<local_image_tag>
```

- Runs the new image in the same pod with the name and commands specified

#### Install

Copy the kubectl-localimage script to your `/usr/local/bin/` directory and make it executable `chmod 755 /usr/local/bin/kubectl-localimage`

