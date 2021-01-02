# Information

Setup of Matomo and MariaDB in Kubernetes. MariaDB is deployed as stateful deployment inside the Cluster. Matomo is connected to MariaDB inside the Cluster. For Matomo we are also using NGINX as proxy pass. This is needed because Matomo does not run at root but as relative path inside Kubernetes. In NGINX we are setting the 'proxy_set_header X-Forwarded-Uri' like described here: https://matomo.org/faq/how-to-install/faq_98/

## Local vs Kubernetes Setup

You can find two folders inside this repository. One setup by using docker-compose locally and one for running in Kubernetes. The Kubernetes setup includes also an NGINX deployment, Matomo will be running mit NGINX inside one POD together. NGINX will be used as Proxy. The difference between the two setups -> locally you can use the port directly without subpath, in Kubernetes Cluster you normally have subpaths. 


# GitHub Projects and Docker Images 

<b> Why Bitnami and why non privileged NGINX? </b>  
We want to have non root containers with restricted rights. You can find all code from the official repositories.

<b> GitHub Projects </b>   
https://github.com/nginxinc/docker-nginx-unprivileged
https://github.com/bitnami/bitnami-docker-matomo
https://github.com/bitnami/bitnami-docker-mariadb

<b> Images in DockerHub </b>   
Matomo Image: bitnami/matomo:4-debian-10  
MariaDB Image: bitnami/mariadb:10.3-debian-10  
NGINX Image: nginxinc/nginx-unprivileged:1.17.5-alpine


# Installation

Steps to install Matomo in Kubernetes:
1. Deploy MariaDB stateful set. It creates the matomo database and add the matomo user. 
2. Deploy Matomo and NGINX.  
   Please make sure that following variable is set `initialInstallation: notInstalled`.   
   Why do we need to set this variable with value `notInstalled`?   
   Matomo is going to install all tables within the database when it's started for the first time. It also creates the `config.ini.php` file where everything which is important for the configuration is stored. If we are trying to create/mount the `config.ini.php`file before the tables are created, Matomo will fail because it needs those tables whithin the database. 
   Why do we need the `config.ini.php` file?  
   The problem with the Bitnami Docker image at the moment is, that we cannot set the `assume_secure_protocol` configuration from the outside. Only `proxy_uri_header`can be set via environment variable. We need both for making sure that Matomo can run behind a proxy. If we do not set the `assume_secure_protocol` we will always getting warnings inside Matomo because it thinks that it does not run under https: https://matomo.org/faq/how-to-install/faq_98/. So both configurations can only be set inside the `config.ini.php` file.  
   So what's the solution at the moment?  
   We install Matomo "normally" and after the "first" installation, we are going to set the `initialInstallation` with value `installed`. In the deployment file you can see that there is an if / else statement, so if the value is set to `installed` config file will be mounted inside the container. 
   What's about the misc / index.html configmap / file?   
   We also have to mount this manually after the "first" installation because the bitnami version of Matomo needs this file also mounted next to the `config.ini.php` file.


# Info for storage classes when using PVC

Normally you have different kind of storage class names depending on which environment your application is running like "fast, slow, manual, ssd, hdd" and so on. In order to see which storage class your provider (locally, on premise or on cloud) provides you can use `kubectl get storageclass` to list all avalailable storage classes.


# Info for missing ingress files / virtual service files

You will not find any ingress or istio files here. This setup was done without any of them. If you want to use ingress files you have to consider what kind of Ingress Controller you want to use like NGINX, Traeffic, KONG and so on. For Istio you need two define your Istio Gateway and also the concerning Virtual Service for the right routing to your application.

Sample of a virtual service configuration would be: 
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: matomo
  namespace: istio-system
gateways:
    - gateway.istio-system.svc.cluster.local
  hosts:
    - "*"
  exportTo:
    - "."
  http:
    - match:
        - uri:
            prefix: /matomo/
        - uri:
            prefix: /matomo
      route:
        - destination:
            host: matomo.YOUR_NAMESPACE.svc.cluster.local
            port:
              number: 80
          weight: 100
```

# Testing

Inside this repository you also find a `hello_world.html` file for testing the tracking feature of Matomo. It will collect information like Device, Browser, Operation System, Country and Action (short overview of amounts of visits etc). If you want to use the file, please change the tracking url `http://Your_Matomo_URL_With_or_Without_Subpath` to your Matomo Instance URL. 