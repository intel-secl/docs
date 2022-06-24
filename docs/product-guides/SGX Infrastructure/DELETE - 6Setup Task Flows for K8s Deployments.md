# Setup Task Flows for K8s Deployments

Setup tasks flows have been updated to have K8s native flow to be more agnostic to K8s workflows. Following would be the flow for setup task. The setup task options are detailed under [Intel Security Libraries Configuration Settings](#intel-security-libraries-configuration-settings) for each service and agent

- User would create a new `configMap`  object with the environment variables specific to the setup task. The Setup task variables would be documented in the Product Guide
- Users can provide variables for more than one setup task
- If variables involve BEARER_TOKEN then user need to delete the corresponding secret using `kubectl delete secret <service_name>-secret -n isecl` , update the new token inside secrets.txt file and re-create the secret using `kubectl create secret generic <service_name>-secret --from-file=secrets.txt --namespace=isecl`, else update the variables in configMap.yml
- Users need to add `SETUP_TASK: "<setup task name>/<comma separated setup task name>"` in the same `configMap`
- Provide a unique name to the new `configMap`
- Provide the same name in the `deployment.yaml` under `configMapRef` section
- Deploy the specific service again with  ` kubectl kustomize . | kubectl  apply -f -`
