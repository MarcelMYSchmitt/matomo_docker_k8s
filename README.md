# Information

Simple setup of Matomo with MariaDB as storage. In this repository you can find a docker-compose.yaml based on the official provided file by Bitnami and own custom Kubernetes deployment files. These deployment files are available as standard files and also as helm charts. 

You can find all relevant information about how to run Matomo in Docker here: https://github.com/bitnami/bitnami-docker-matomo


---


## Info for storage classes when using PVC

This setup was made on a local machine where Docker Desktop for Windows was running. So in order to use PVCs the storage class which was used is called "hostpath". Normally you have different kind of storage class names depending on which environment your application is running like "fast, slow, manual, ssd, hdd" and so on. 

In order to see which storage class your provider (locally, on premise or on cloud) provides you can use `kubectl get storageclass` to list all avalailable storage classes.


--- 


## Info for missing ingress files / virtual service files

You will not find any ingress or istio files here. This setup was done without any of them. For testing purpose you can use port-forwarding like `kubectl -n test port-forward service/matomo 1234:443`. If you want to use ingress files you have to consider what kind of Ingress Controller you want to use like NGINX, Traeffic, KONG and so on. For Istio you need two define your Istio Gateway and also the concerning Virtual Service for the right routing to your application.


---


## Info for security

You can find for MariaDB and Matomo the corresponding network policies for allowing traffic towards the services. Outgoing traffic is not allowed and should be avoided. 


---


## To Dos

Extend deployment files for readiness and livenessprobes, create helm charts based on them, extend readme! 