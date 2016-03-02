# Kubernetes in AWS on CoreOS

Download the kube-aws executable:

OS X: https://github.com/coreos/coreos-kubernetes/releases/download/v0.4.0/kube-aws-darwin-amd64.tar.gz
Linux: https://github.com/coreos/coreos-kubernetes/releases/download/v0.4.0/kube-aws-linux-amd64.tar.gz

```
$ tar -xf <file> kube-aws
```

Make sure kube-aws is somewhere on your path.

## Configuring the cluster

Note: I have supplied a default cluster.yaml file, I initially had some trouble with it by not specifying an availabilityZone. It will select REGIONa automatically, and that AZ is over-provisioned in us-east-1 and us-west-1.

## Starting up the cluster
```
kube-aws up
```

Once the cluster has been created, there will be a clusters folder with the PKI needed to run the cluster. Additionally, it will generate a kubeconfig, that we will reference in commands later on.

## Operating the cluster
```
kubectl --kubeconfig=clusters/$clustername/kubeconfig get nodes
kubectl --kubeconfig=clusters/$clustername/kubeconfig get rc
kubectl --kubeconfig=clusters/$clustername/kubeconfig get services
```

## Creating our guestbook application
Note: I object to the use of master/slave nomenclature here, but I don't have access to the source of the guestbook frontend, so I couldn't change it.

### Create the redis primary replication controller and service
```
kubectl --kubeconfig=clusters/$clustername/kubeconfig create -f examples/guestbook/redis-master-controller.yml
kubectl --kubeconfig=clusters/$clustername/kubeconfig create -f examples/guestbook/redis-master-service.yml
```

### Create the redis follower replication controller and service
```
kubectl --kubeconfig=clusters/$clustername/kubeconfig create -f examples/guestbook/redis-slave-controller.yml
kubectl --kubeconfig=clusters/$clustername/kubeconfig create -f examples/guestbook/redis-slave-service.yml
```

### Create the frontend replication controller and service
```
kubectl --kubeconfig=clusters/$clustername/kubeconfig create -f examples/guestbook/frontend-controller.yml
kubectl --kubeconfig=clusters/$clustername/kubeconfig create -f examples/guestbook/frontend-service.yml
```

Setting up the frontend service takes a few minutes, but it creates an Elastic Load Balancer in the background. I couldn't find a way for k8s to expose it, you have to look up the ELB url in your AWS console.