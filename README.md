# Information

<b> STILL IN PROGRESS AND NOT READY TO BE USED YET! </b>

Simple setup of Matomo with MariaDB as storage. In this repository you can find a docker-compose.yaml and custom Kubernetes deployment files. These deployment files are available as standard files and also as helm charts. 
In the docker-compose yaml you can find a MariaDB database image from Bitnami. They are also providing a Matomo Docker Image. You can find all relevant information about these two Docker Images here: https://github.com/bitnami/bitnami-docker-matomo

When I first set everything up I tried to reuse everything from Bitnami...but now I'm only using the MariaDB Docker Image. Bitnami provides this image as runnable non-root Image. Matomo however is no runnable non-root Image (take a look at Security below). 

---


## Info for storage classes when using PVC

This setup was made on a local machine where Docker Desktop for Windows (whith local Kubernetes Cluster) was running. So in order to use PVCs the storage class which was used is called "hostpath". Normally you have different kind of storage class names depending on which environment your application is running like "fast, slow, manual, ssd, hdd" and so on. 

In order to see which storage class your provider (locally, on premise or on cloud) provides you can use `kubectl get storageclass` to list all avalailable storage classes.

Attention: You can only find the stateful MariaDB Deployment in the 'classic' deployment files. In the helm chart it's just a stateless / simple deployment. If you also want to have the helm chart as statefull deployment please change it!

--- 


## Info for missing ingress files / virtual service files

You will not find any ingress or istio files here. This setup was done without any of them. For testing purpose you can use port-forwarding like `kubectl -n YOUR.NAMESPACE port-forward service/matomo 1234:443`. If you want to use ingress files you have to consider what kind of Ingress Controller you want to use like NGINX, Traeffic, KONG and so on. For Istio you need two define your Istio Gateway and also the concerning Virtual Service for the right routing to your application.

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

---


## Security

We distinguish among 'Traffic'  and 'runnable Non-Root Images'. 


### Traffic

You can find for MariaDB and Matomo the corresponding network policies for allowing traffic towards the services. Outgoing traffic is not allowed and should be avoided.  

### Non-Root

When you deploy Images in your Kubernetes Cluster you can deploy them as `normal` containers without regarding any security purpose. But normally you have some kind of security guidelines to consider for running PROD Kubernetes Clusters like to make sure that PODs are not allowed to be run with higher privileges. 

Bitnami provides MariaDB as non-root Image: https://hub.docker.com/r/bitnami/mariadb  
When you take a look into the Dockerfile you can see that the USER is set to `1001`: https://github.com/bitnami/bitnami-docker-mariadb/blob/10.5.6-debian-10-r14/10.5/debian-10/Dockerfile  

The Matomo Image provided from Bitnami does not support changing the permissions/privilege of a USER. If you google you can find an Image provided by Wodby: https://hub.docker.com/r/wodby/matomo  
Inside of the Dockerfile you can also find code lines where the permissions where changed by using `CHMOD`: https://github.com/wodby/matomo/blob/master/Dockerfile

Beeing lazy, I decided to reuse the Image from Wodby for my repository here :) 
