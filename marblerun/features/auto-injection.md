# Kubernetes integration

MarbleRun provides its data-plane configuration through Kubernetes resource definitions. For this, like regular service meshes, MarbleRun uses Kubernetes' [admission controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook).

MarbleRun optionally injects [tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) and [resources](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) for its SGX device plugin. See the [Kubernetes deployment](deployment/kubernetes.md#sgx-device-plugin-on-kubernetes) section for more information.

You can enable auto-injection of the data-plane configuration for a namespace using the MarbleRun CLI:

```bash
marblerun namespace add NAMESPACE [--no-sgx-injection]
```

This will add the label `marblerun/inject=enabled` to the chosen namespace and allow the admission webhook to intercept the creation of deployments, pods, etc. in that namespace.
The flag `--no-sgx-injection` disables the label `marblerun/inject-sgx`. This is useful when using your own SGX device plugin.

## The Marbletype label

In MarbleRun, marbles (i.e, secure enclaves) are defined in the [manifest](workflows/define-manifest.md). You need to reference marbles in your Kubernetes resource description as follows using the `marblerun/marbletype` label:

```javascript
{
    "Marbles": {
        "voting-svc": {
    // ...
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting
  namespace: emojivoto
  labels:
    app.kubernetes.io/name: voting
    app.kubernetes.io/part-of: emojivoto
    app.kubernetes.io/version: v1
    marblerun/marbletype: voting-svc
```

We use this label to map Kubernetes Pods to MarbleRun Marbles.
When you deploy your application, MarbleRun will read out this label's value, `voting-svc` in the example above.
It will check the Marbles section of your manifest for an entry with the same name.
If such entry is present in the manifest, the Pod is provided with the particular configuration for this Marble.
If no such entry exists or a valid `marblerun/marbletype` label is missing, the Pod's creation is rejected.

## Injected environment variables

The webhook will inject the following environment variables into each container of a pod:

* `EDG_MARBLE_TYPE`:  The value of the `marblerun/marbletype` label
* `EDG_MARBLE_COORDINATOR_ADDR`:  The address of the MarbleRun Coordinator running on the cluster
* `EDG_MARBLE_DNS_NAMES`:  DNS names of the pod derived from marbletype and namespace: `marbletype, marbletype.namespace, marbletype.namespace.svc.cluster.local`
* `EDG_MARBLE_UUID_FILE`:  The mounted UUID of the Marble

If an environment variable is already set before the webhook handles the creation request, the variable will not be overwritten and the custom value is used instead.
