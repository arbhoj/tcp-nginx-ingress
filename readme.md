# Layer 4 Edge Termination using Nginx Proxy on k8s #

## Overview ##
This document shows a method of using a Nginx Proxy server to perform layer 4 edge termination for a service running on Kubernetes. It uses a redis server as an example. This is a common scenario when the app/container itself does not provide TLS but all data coming from outside to the k8s cluster needs to be encrypted. 

Note: To keep things simple, just a single worker node has been shown with a single redis pod and a single nginx proxy pod but it can be scaled to multiple nodes/pods. 


## High Level Design and Flow ##

A client connects to the redis instance using redli (which is a command line tool that can be used to connect to a remote redis instance). The client uses the ip and port of the loadbalancer. The loadbalancer and it’s listeners are automatically created when a service of type LoadBalancer is created in k8s. In the above example a listener on load-balancer port 9999 is created that routes traffic to k8s worker nodes on port 32548 (this is the nodePort assigned to the loadbalancer service). This is the nodePort that routes traffic to the nginx-tcp-router Pod which has a configuration in the form of tcp-ingress Configmap that uses nginx’s streams to terminate TLS and proxy traffic to the myred service, which in turn routes traffic to the redis pod.


## Implementation Details ##

![High Level Flow](https://github.com/arbhoj/tcp-nginx-ingress/blob/master/Layer4-Edge-Termination.png)

The solution leverages the following pieces:
1. nginx Deployment – This deploys one or more nginx pods that load:
   a. nginx.conf from the nginx-config configmap
   b. tcp-ingress.conf from tcp-ingress configmap
   c. nginx-cert secret to host the ca.key, server.crt and server.key to use for TLS
2. nginx-config Configmap – This contains: 
   a. Contents of the nginx.conf to the used by the nginx proxy. 
   b. A “stream” section with the certificate details with reference to include any configs loaded into the “/config” dir. This minimizes the amount of lines that need to be added to in tcp-ingress Configmap for each service port
3. tcp-ingress Configmap – This contains:
   a. The “server” and the “upstream” portions of the “streams” directive.
4. nginx-cert secret
5. nginx-tcp service that defines a service of type loadbalancer
6. redis deployment to deploy a redis instance
7. myred service that points to the redis pod

Clone this repo into the machine that is configured to connect to a k8s cluster. CD to the package dir and run “kubectl apply -f .



