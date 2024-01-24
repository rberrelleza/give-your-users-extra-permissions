# Use clusterRoles to give your users extra permissions

Okteto's default permission model gives every user `cluster-admin` access to their namespaces. This means that they can perform any `namespace-scoped` operation on their own namespaces, such as create deployments, update secrets, etc. This is meant to facilitate the development experience, while protecting your clusters and other users. 

If your application requires extra permissions, or you want to give your users wider permissions than Okteto's default, you can do this via the [`globalClusterRole`](https://www.okteto.com/docs/reference/helm-chart-values/#globalclusterrole) configuration setting in Okteto. 


## Example

The included example is an API that returns the number of nodes in your cluster. It uses the Kubernetes API to retrieve this information.  If you run the included sample on your Okteto cluster, and then call the API, the call will fail with something similar to the message below. This is because `cluster-admin` doesn't have permissions to access the `cluster-scoped` Nodes API. 

```
> curl http://hello-world.okteto.example.com
nodes is forbidden: User "system:serviceaccount:pdb:default" cannot list resource "nodes" in API group "" at the cluster scope
```

In order to give the user permission to do this, you need to do the following:
1. Using a kuberentes context that has cluster permissions, create the role: `kubectl apply -f cluster-role/clusterrole.yaml`
2. Update Okteto's helm configuration file to include the clusterRole we just created:
    ```
    license: ....
    subdomain: dev.example.com
    auth: ...
    globalClusterRole: custom-role-with-nodes-access
    ```
3. Apply the configuration: `helm upgrade okteto okteto/okteto -f config.yaml --namespace=okteto --version 1.16.0` 

Once the upgrade is completed, Okteto will update all the user accounts with the updated permissions. At this point, you should be able to call the API and get a response similar to this:

```
> curl http://hello-world.okteto.example.com
Hello world, your cluster has 3 nodes!
```