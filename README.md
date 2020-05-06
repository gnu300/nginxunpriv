# nginxunpriv

You can either build the image based on the Dockerfile on your own or you can pull it from quay.io: `docker pull quay.io/gnu300/nginxunpriv`

This Dockerfile is based on the official Nginx Inc unprivileged Docker Alpine image https://github.com/nginxinc/docker-nginx-unprivileged .
The purpose of this "yet another nginx image" is solely for debugging. Therefore i added additionaly to the official Dockerfile some best practices for an OpenShift environment and debugging tools. 

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

## nginx.conf

For my purpose i just include the nginx.conf with COPY in the Dockerfile.
If you want to overwrite my config with a Kubernetes/OpenShift ConfigMap here are the corresponding yaml snippets.

    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: nginx
      namespace: your-namespace
    data:
      nginx.conf: |
      worker_processes 2;    

      error_log /var/log/nginx/error.log;
      pid /tmp/nginx.pid;    

      events {}    

      http {
        log_format nginx '"$time_local" client=$remote_addr:$remote_port '
                      'method=$request_method request="$request" '
                      'request_length=$request_length '
                      'status=$status bytes_sent=$bytes_sent '
                      'body_bytes_sent=$body_bytes_sent '
                      'referer=$http_referer '
                      'user_agent="$http_user_agent" '
                      'ssl="$ssl_protocol/$ssl_cipher" '
                      'session="$ssl_session_id" '
                      'upstream_addr=$upstream_addr '
                      'upstream_status=$upstream_status '
                      'request_time=$request_time '
                      'upstream_response_time=$upstream_response_time '
                      'upstream_connect_time=$upstream_connect_time '
                      'upstream_header_time=$upstream_header_time';    

        access_log  /var/log/nginx/access.log  nginx;    

        server_tokens off;    

        server {
            listen 8080;    

            location /health {
                access_log off;
                allow 127.0.0.1;
                deny all;    

                limit_except GET {
                  deny all;
                    }
                return 200 "healthy\n";
                    }
                }    
    

           }
    --
    kind: Pod
    apiVersion: v1
    metadata:
      name: nginx-pod
      namespace: your-namespace
    spec:
      restartPolicy: Always
      containers:
          name: nginx-container
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
          imagePullPolicy: Always
          volumeMounts:
            - name: nginx
              mountPath: /etc/nginx/nginx.conf
              subPath: nginx.conf
          image: 'xyz'
      volumes:
        - name: nginx
          configMap:
            name: nginx
            defaultMode: 420



## Note
In order to tcpdump to work properly it needs greater privileges to dump the network traffic in the pod. This should be only done if you know what you are doing but in any case execute with care:

`oc adm policy add-scc-to-user anyuid -z default -n $PROJECT --as=system:admin`

**IMPORTANT**
This is not for production environments and could cause a potential security issue, because any pod can run as root so be careful and only implement this in testing environments and for debugging purpose.
