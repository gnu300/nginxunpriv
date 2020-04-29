# nginxunpriv

This Dockerfile is based on the official Nginx Inc unprivileged Docker Alpine image https://github.com/nginxinc/docker-nginx-unprivileged . The purpose of this "yet another nginx image" is solely for debugging. Therefore i added additionaly to the official Dockerfile some best practices for an OpenShift environment and debugging tools. 

## Additonal packages
* python3
* py3-netifaces
* py3-prettytable
* py3-certifi
* py3-chardet
* py3-future
* py3-idna
* py3-netaddr
* py3-parsing
* py3-six
* nmap
* nmap-scripts
* curl
* tcpdump
* bind-tools
* jq
* nmap-ncat
* bash

## Note
In order to tcpdump to work properly it needs greater privileges to dump the network traffic in the pod. This should be only done if you know what you are doing but in any case execute with care:

`oc adm policy add-scc-to-user anyuid -z default -n $PROJECT --as=system:admin`

**IMPORTANT**
This is not for production environments and could cause a potential security issue, because any pod can run as root so be careful and only implement this in testing environments and for debugging purpose.
