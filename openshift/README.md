Infinispan + Openshift  example
===============================

This repo contains a sample of creating an Infinispan cluster on Openshift, using the ConfigMap API, with and without KUBE_PING

### Give permission to access kube api

```
./oc login -u system:admin
./oc policy add-role-to-user view system:serviceaccount:myproject:default -n myproject
./oc login -u developer -p developer
```

### Creating the cluster

```
./oc create -f infinispan.yaml
```

### Scaling

```
./oc scale --replicas=4 dc/infinispan-server
```

### Deleting all objects

```
./oc delete all -l "app=infinispan-server"
```

### Creating the config map

The file ```my-config.xml``` is a customized Infinispan configuration with a custom distributed cache named ```custom-cache```. 
To create a Config Map from this file, execute the command:

```
kubectl create configmap infinispan-config --from-file=my-config.xml
```

the file can be dumped with:

```
kubectl get configmaps infinispan-config -o yaml
```

### Creating the cluster

The file ```infinispan.yaml``` will create an Infinispan cluster with discovery done using the JGroups Gossip, and will
use the configuration from the Config Map named ```infinispan-config```. To import it on Kubernetes, use:


```
kubectl create -f infinispan.yaml
```

To cleanup, use the label applied to all objects:

```
kubectl delete deployments,services,pods -l "app in (infinispan)"
kubectl delete configmap/infinispan-config
```

### Checking the environment

To check the numbers of members in each of the Infinispan pods, run the following command:

```
for pod in $(kubectl get pods | grep infinispan-server | awk '{print $1}'); do kubectl exec -it $pod -- infinispan-server/bin/ispn-cli.sh -c '/subsystem=datagrid-infinispan/cache-container=clustered:read-attribute(name=members)'; done
```

### Check the custom cache

To verify that the custom config was actually created in all nodes, the command below print the full cache configuration:

```
for pod in $(kubectl get pods | grep infinispan-server | awk '{print $1}'); do kubectl exec -it $pod -- infinispan-server/bin/ispn-cli.sh -c '/subsystem=datagrid-infinispan/cache-container=clustered/configurations=CONFIGURATIONS/distributed-cache-configuration=custom-cache:read-resource(recursive=true)'; done

```
