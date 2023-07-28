## Run a Replicated Stateful Application

### 1. Preconditions
- Create a K8S cluster (Please check -> https://github.com/mehmetmgrsl/k8s-prometheus-grafana)

### 2. Deploy MySQL

- ```kubectl apply -f mysql-cm.yaml``` # Create the ConfigMap
- ```kubectl apply -f mysql-svcs.yaml``` # Create Services
- ```kubectl apply -f mysql-statefulset.yaml``` # Create the StatefulSet

### Check the status of PODs

```
>>> kubectl get pods

NAME      READY   STATUS    RESTARTS   AGE
mysql-0   2/2     Running   0          8m25s
mysql-1   2/2     Running   0          5m6s
mysql-2   2/2     Running   0          115s
```

### Sending client traffic 

1. Send a query to writer

```
kubectl run mysql-client --image=mysql:5.7 -i --rm --restart=Never --\
  mysql -h mysql-0.mysql <<EOF
CREATE DATABASE test;
CREATE TABLE test.messages (message VARCHAR(250));
INSERT INTO test.messages VALUES ('hello');
EOF
```

2. Send a query to reader pool

```
kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-read -e "SELECT * FROM test.messages"

+---------+
| message |
+---------+
| hello   |
+---------+
```

### Delete PODs

```kubectl delete pod mysql-1```

The StatefulSet also recreates Pods if they're deleted.

The StatefulSet controller notices that no ```mysql-1``` Pod exists anymore, and creates a new one with the same name and linked to the same PersistentVolumeClaim.

```
kubectl delete pod mysql-1
pod "mysql-1" deleted

kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mysql-0   2/2     Running   0          18m
mysql-1   2/2     Running   0          37s # created again
mysql-2   2/2     Running   0          12m
```

### Resources
1. [Run a Replicated Stateful Application
](https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/)
2. [Deploy a Production Database in Kubernetes](https://www.youtube.com/watch?v=UDXnyh0vtXw)