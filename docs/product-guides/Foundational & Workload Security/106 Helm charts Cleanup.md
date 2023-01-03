# Steps for Helm chart Cleanup

To uninstall a chart
```
helm uninstall <release-name> -n <namespace>
```

To list all the helm chart deployments 
```
helm list -A
```

# Cleanup steps that needs to be done for a fresh deployment
   * Uninstall all the chart deployments.
   * Cleanup the data at NFS mount and trustagent data mount on each nodes (/etc/trustagent, /var/log/trustagent)
   * Remove all objects(secrets, rbac, clusterrole, service account) related namespace related to deployment ```kubectl delete ns <namespace>```. 
   * For charts Trusted-Workload-Placement and Trusted-Workload-Placement-Cloud-Service-Provider, ISecl-Scheduler should be disconnected from K8s base scheduler. This can be done by configuring in manifest of kube-scheduler as mentioned below, by commenting the *--config* option
   ```
   containers:
   - command:
     - kube-scheduler
     - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
     - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
     - --bind-address=127.0.0.1
     - --kubeconfig=/etc/kubernetes/scheduler.conf
     - --leader-elect=true
     #- --config=/opt/isecl-k8s-extensions/kube-scheduler-configuration.yml
   ```
   * Restart Kubelet which restart all the k8s services including kube base scheduler
   ```console
	   systemctl restart kubelet
   ```

**Note**: 
```
Before redeploying any of the chart please check the pv and pvc of corresponding deployments are removed. Suppose if you want to redeploy aas, make sure that aas-logs-pv, aas-logs-pvc, aas-config-pv, aas-config-pvc, aas-db-pv, aas-db-pvc, aas-base-pvc are removed successfully.
Command: `kubectl get pvc -n <namespace>` && `kubectl get pv -n <namespace>`
```


