# DDSL Wordpress Kubernetes Deployment

This repository provides the details of the kubernetes deployment of a three-tier wordpress application that can be used for benchmarking several auto-scaling algorithms.
This repository provides you with the `yaml` files needed to deploy the structure and get the kubernetes deployment up and running.

## Deployment Structure

This three-tier wordpress application consists of `MySql 5.6` as the database, `Wordpress with PHP 7.3 FPM` as the application server and `Nginx 1.7.9` as the web server.
 
 ## Deployment Procedure

 - Before setting up the kubernetes cluster, we need an NFS server to serve the static files used by Wordpress and Nginx. All instances of our web server and application server will use this file server. 
 
 - Follow [This Tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-16-04) for setting up the NFS server. In `/etc/exports`, use the following as the permission for the `/var/nfs/general`:

```sh
(rw,sync,no_subtree_check,no_root_squash)
```

- Don't forget to allow NFS server access to your cluster either by setting your cluster's IP range in `/etc/exports` or by setting the client IP to `*` to allow any host to connect and limit the access to the NFS server via your firewall.

<aside class="notice">
You need to open ports 111 and 2049 for both TCP and UDP for NFS to function properly.
</aside>


- Note your NFS server IP address since you will have to input this into your yaml files. We will call this value `NFS_SERVER_IP` from now on.

- Make a copy of the wordpress yaml template:

```sh
cp ./yaml/wordpress-deployment-template.yaml ./yaml/wordpress-deployment.yaml
```

- For security reasons, update `lines 57-64` with new values from [this address](https://api.wordpress.org/secret-key/1.1/salt/).

- Upload `line 137` with the IP address of your NFS server.

- In case you want to use your private docker registry for faster pulls from your cluster, first upload the Wordpress FPM docker image to your private registry using `docker pull wordpress:php7.3-fpm-alpine`, then `docker tag wordpress:php7.3-fpm-alpine -t YOUR_PRIVATE_DOCKER_REGISTRY_ADDRESS/wordpress:php7.3-fpm-alpine` and then `docker push YOUR_PRIVATE_DOCKER_REGISTRY_ADDRESS/wordpress:php7.3-fpm-alpine`. After doing this step, update `line 179` of `wordpress-deployment.yaml` with `YOUR_PRIVATE_DOCKER_REGISTRY_ADDRESS/wordpress:php7.3-fpm-alpine` as the docker image address.

- Deploy the structure to your kubernetes cluster using [Kustomization](https://kustomize.io/):

```sh
kubectl apply -k ./yaml/
```

- It should take about 5 minutes for the whole deployment to get to the `Running` state. To monitor the progrss, you can use `kubectl get events -w`, `kubectl get deploy -w`, or `kubectl get pods -w`. After the service is up and running, you can use `kubectl get svc` to get a list of services along the IP of the load balancer assigned to them:

```sh
NAME              TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)          AGE
kubernetes        ClusterIP      X.X.X.X        <none>           443/TCP          14d
wordpress         ClusterIP      X.X.X.X        <none>           9000/TCP         14d
wordpress-mysql   ClusterIP      None           <none>           3306/TCP         14d
wordpress-nginx   LoadBalancer   X.X.X.X        X.X.X.X          80:32349/TCP     14d
```

The external IP in the `EXTERNAL-IP` column of `wordpress-nginx` is the IP you can use to access your wordpress setup.
