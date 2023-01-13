# Documentation

<https://medium.com/@emilfabrice/deploy-microk8s-kubernetes-on-ubuntu-with-ansible-844f22e154a0>

Additionally, as at the time of writing installation of `metallb` does not work via Ansible, it should be done manually on the controll node:

```bash
microk8s enable metallb:10.0.2.200-10.0.2.220
```

## Add sample Helm application

Follow steps [here](https://artifacthub.io/packages/helm/bitnami/nginx) with some ammendments:

```bash
# install the bitnami chart repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# (optional) verify the repo is present
helm repo ls
#NAME    URL
#bitnami https://charts.bitnami.com/bitnami

# install the application
helm install webserver bitnami/nginx
```

As per the instructions displayed on the screen, to access the deployed webserver, you should execute:

```bash
    export SERVICE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].port}" services webserver-nginx)
    export SERVICE_IP=$(kubectl get svc --namespace default webserver-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo "http://${SERVICE_IP}:${SERVICE_PORT}"
    #http://10.0.2.200:80
```

At this point the webserver should be accessible at the above ip/port combination.

## Helm with values

The previous example was deploying Nginx with the default values. Let's see how can we deploy a different application with specified values.

So steps are:

- update the `wordpress-values.yml`

- install this helm chart

```bash
 helm install wordpress bitnami/wordpress --values=helm/wordpress-values.yml
```

- Find the required values for the connection
```bash

# get the connection details
 export SERVICE_IP=$(kubectl get svc --namespace default wordpress --include "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

# if above fails, just do
kubectl get service wordpress
## NAME        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
## wordpress   LoadBalancer   10.152.183.65   10.0.2.201    80:31311/TCP,443:32060/TCP   3m31s
# and use the extenal IP returned for connection, so for example: http://10.0.2.201/admin in this case

# get the password with
echo Password: $(kubectl get secret --namespace default wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)

```
