## Workload POD Launch Validation

### Launch Workload for SGX

- Create below sample sgx_pod.yml file for nginx workload and add SGX labels as node affinity to it.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-sgx
  labels:
    name: nginx-sgx
spec:
  affinity:
    nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: SGX-Enabled
           operator: In
           values:
           - "true"
         - key: EPC-Memory
           operator: In
           values:
           - "2.0GB"
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

- Launch the pod by running below command:

```shell
kubectl apply -f sgx_pod.yml
```

* Check pod status using

```shell
kubectl get pod -l name=nginx-sgx
```



### Launch Workload for FS

- Create below sample `fs_pod.yml` file for nginx workload and add trusted labels as node affinity to it.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-fs
  labels:
    name: nginx-fs
spec:
  affinity:
    nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: isecl.trusted
           operator: In
           values:
           - "true"
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

- Launch the pod by running below command:

```shell
kubectl apply -f fs_pod.yml
```

* Check pod status using

```shell
kubectl get pod -l name=nginx-fs
```
