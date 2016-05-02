# ROS on Kubernetes

This repository contains a simple example of how to run ROS nodes on Kubernetes.
It creates 3 pods: a master, a talker, and a listener that communicate on a ROS topic.
The whole point of the experiment was to find out the correct configuration of 
Kubernetes pods and services that allow ROS nodes to discover and talk to each other.

Both talker and listener have to register to the master in order to communicate on a topic.

We're using the excellent "official" ROS containers from Docker Hub: https://hub.docker.com/_/ros/

A blog article about this implementation is available here: https://blog.zhaw.ch/icclab/challenges-with-running-ros-on-kubernetes/

## TL;DR: What worked for us

- We use one pod per ROS node and we wanted the nodes to be able to talk to each other seamlessly
- Kubernetes services create A Records in the internal Kubernetes DNS, so we 
thought this was a good way to let nodes find each other
- It turns out that "regular" services can only be declared in association with 
one or more ports, while ROS nodes use dynamic ports and let the master do the 
matchmaking
- Luckily, Kubernetes also supports the notion of "Headless" services (http://kubernetes.io/docs/user-guide/services/#headless-services)
- With a headless service, each pod associated with a service gets its own DNS 
entry rather than the service getting a Virtual IP and relaying on Kube-Proxies 
and a specific port.

## Prerequisites

You'll need a Kubernetes cluster, and kubectl correctly configured to talk to it

## Running

Clone the repo:

    git clone https://github.com/gtoff/ros_on_kubernetes.git

Create all pods and services ("ros-on-kubernetes" is the directory you just cloned)

    kubectl create -f ros_on_kubernetes

Check that pods and services are created

    kubectl get all

Connect to the listener pod and listen to the talker:

    kubectl exec -it listener bash
    root@listener:/# source /opt/ros/indigo/setup.bash
    root@listener:/# rostopic echo /my_topic

You should see the messages coming through the topic:
```
root@listener:/# rostopic echo /my_topic
data: SPLab+ICClab
---
data: SPLab+ICClab
---
data: SPLab+ICClab
```

## Troubleshooting

Actually the listener node should already be logging the messages on standard 
output, and it should be possible to see them with:

     kubectl logs listener

However, I'm suspecting there are some run conditions depending on the creation order of the pods / services.
Still needs a bit of investigation.

