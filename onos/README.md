# Testing ONOS Clustering Under Kubernetes

**FIRST**: Follow directions at `https://github.com/davidkbainbridge/k8s-playground` to set up a
kubernetes test environment. Use the `start-weave` command to use weave networking (weave is tested
to work for the example environment).

### Create Services
Two services are create, one for ONOS and the other for a service that will serve the cluster configuration to ONOS.

On one of the vagrant machines that are part of the cluster execute the following commands.

```
kubectl create -f onos-service.yml -f cluster-config-service.yml
```

you can see the services created by

```
$ kubectl get svc -o wide
NAME                     CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE       SELECTOR
cluster-config-service   10.100.243.174   <none>        80/TCP                       2h        app=cluster-config-service
fabric-controller        10.96.128.54     <none>        8181/TCP,8101/TCP,6653/TCP   2h        app=fabric-controller
kubernetes               10.96.0.1        <none>        443/TCP                      3h        <none>kubectl get svc -o wide
```

### Create the Cluster Configuraiton Instance

```
kubectl create -f cluster-config-deployment.yml
```

### Create Initial Cluster Configuration

Use the following command to retrieve the `NODE` on which the cluster configuration service is running

```
$ kubectl get po -o wide
NAME                                     READY     STATUS    RESTARTS   AGE       IP          NODE
cluster-config-service-902951554-njkbt   1/1       Running   0          2h        10.40.0.2   k8s3
```

On the specified node (vagrant VM) execute the following command to create the empty configuraiton file

```
sudo /vagrant/onos/onos-gen-partitions -n 3 /data/config.json $(kubectl get po -l app=fabric-controller --template='{{ range $key, $value := .items }}{{$value.status.podIP}} {{end}}')
```

you can verify the file by

```
$ cat /data/config.json
{
    "nodes": [],
    "name": 3820012610,
    "partitions": [
        {
            "id": 1,
            "members": []
        },
        {
            "id": 2,
            "members": []
        },
        {
            "id": 3,
            "members": []
        }
    ]
}
```

### Create Instances

Create the initial ONOS instance

```
kubectl create -f onos-deployment.yml
```

It may take a while for the instance to start as the image has to be downloaded. Using `kubectl get po` you can
verify when the instance is `READY`.

### Checking Node Status

Once the instance is ready you can ssh into the instance using the `CLUSTER-IP` from the `kubectl get svc -o wide` command.

```
ssh -p 8101 karaf@10.96.128.54
```

and issue the nodes command. with an empty cluster configuraiton you should only get

```
onos> nodes
Service org.onosproject.cluster.ClusterAdminService not found
```

to convert the cluster configration to a single node cluster config use the following command

```
sudo /vagrant/onos/onos-gen-partitions -n 3 /data/config.json $(kubectl get po -l app=fabic-controller --template='{{ range $key, $value := .items }}{{$value.status.podIP}} {{end}}')
```

This sould make the `nodes` command work in ONOS, but the state will stay active.

### Scaling

you can change the number of ONOS instances using the following command

```
kubectl scale --replicas=3  -f deployment.yml
```

and when the replicates are started (see `kubectl get po`) you can use the following command to update the cluster configuration

```
sudo /vagrant/onos/onos-gen-partitions -n 3 /data/config.json $(kubectl get po -l app=fabic-controller --template='{{ range $key, $value := .items }}{{$value.status.podIP}} {{end}}')
```

the cluster never forms or works
