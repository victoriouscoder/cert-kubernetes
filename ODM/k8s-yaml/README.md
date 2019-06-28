# Install IBM Operational Decision Manager with the Kubernetes CLI

Before you install make sure that you have prepared your environment. For more information, see [Preparing to install ODM for production](https://www.ibm.com/support/knowledgecenter/SSYHZ8_19.0.x/com.ibm.dba.install/k8s_topics/tsk_preparing_odmk8s.html) as well as [Customizing ODM for production](https://www.ibm.com/support/knowledgecenter/SSYHZ8_19.0.x/com.ibm.dba.install/k8s_topics/tsk_install_odm.html).

Refer to the top repository [readme](../../README.md#step-2-download-a-product-package-from-ppa-and-load-the-images) to find instructions on how to push and tag the product container images to your Docker registry.

1. If Helm is not installed in your Kubernetes cluster, install [Helm 2.9.1](https:/github.com/helm/helm/releases/tag/v2.9.1).

2. Download the `ibm-odm-prod-2.2.0.tgz` Helm chart.
   - [ibm-odm-prod-2.2.0.tgz](ibm-odm-prod-2.2.0.tgz) for Operational Decision Manager 8.10.2

3. Create a chart YAML template file with the default configuration parameters by using the following command. The `--name` argument sets the name of the release to install.

   ```console
   $ helm template \
     --name my-odm-prod-release \
     /path/to/ibm-odm-prod-2.2.0.tgz > generated-k8s-templates.yaml
   ```

4. Install `my-odm-prod-release` with the default configuration by using the following command.

   ```console
   $ kubectl apply -f generated-k8s-templates.yaml
   ```
   The package is deployed asynchronously in a matter of minutes, and is composed of several services.

   > **Note**: You can check the status of the pods that you created:
   ```console
   $ kubectl get pods
   NAME                                                READY   STATUS    RESTARTS   AGE
   my-odm-prod-release-dbserver-***                    1/1     Running   0          44m
   my-odm-prod-release-odm-decisioncenter-***          1/1     Running   0          44m
   my-odm-prod-release-odm-decisionrunner-***          1/1     Running   0          44m
   my-odm-prod-release-odm-decisionserverconsole-***   1/1     Running   0          44m
   my-odm-prod-release-odm-decisionserverruntime-***   1/1     Running   0          44m
   ```

   The release is an instance of the `ibm-odm-prod` chart. All of the Operational Decision Manager components are now running in a Kubernetes cluster.

   To verify a deployment, go back to the [Post installation steps](../README.md#post-installation-steps).

## Customize a Kubernetes release of Operational Decision Manager

Refer to the [ODM for production Certified Kubernetes parameters](https://www.ibm.com/support/knowledgecenter/SSYHZ8_19.0.x/com.ibm.dba.ref/k8s_topics/ref_parameters_prod.html) for a complete list of values that you can configure.

### To customize the install with --set key=value arguments

Using Helm, you can specify each parameter with a `--set key=value` argument in the `helm template` command.

For example:
```console
$ helm template --name my-odm-prod-release \
  --set internalDatabase.databaseName=my-db \
  --set internalDatabase.user=my-user \
  --set internalDatabase.password=my-password \
  /path/to/ibm-odm-prod-2.2.0.tgz
```

### To customize the helm install with a YAML file

It is also possible to use a custom-made .yaml file to specify the values of the parameters when you install the chart.
For example:

```console
$ helm template --name my-odm-prod-release -f myvalues.yaml /path/to/ibm-odm-prod-2.2.0.tgz
```

> **Tip**: Refer to the [`sample-values.yaml`](../configuration/sample-values.yaml) file to find the default values used by the `ibm-odm-prod` chart.

## Upgrade a release

1. [Download the latest PPA file from IBM Passport Advantage and load the new images.](../README.md#step-2-download-a-product-package-from-ppa-and-load-the-images)

3. Delete the odm-test pod

   ```console
   $ kubectl delete pod my-prod-release-odm-test
   ```

2. Create a new chart YAML template file.

   > **WARNING**: You must reuse the same `--set key=value` arguments and/or values.yaml file that were specified during the previous installation or the configuration will be reset to its default values.

   ```console
   $ helm template \
     --name my-odm-prod-release \
     --set key=value \
     -f myvalues.yaml \
     /path/to/ibm-odm-prod-2.2.0.tgz > generated-k8s-templates-upgrade.yaml
   ```

4. Apply this new template in Kubernetes.

   ```console
   $ kubectl apply -f generated-k8s-templates-upgrade.yaml
   ```

   > **Note**: The Persistent Volume Claim is not recreated. You can ignore the message: `The PersistentVolumeClaim "my-odm-prod-release-pvclaim" is invalid: spec: Forbidden: is immutable after creation except resources.requests for bound claims`

5. Verify that the version of Decision Center and the Decision Server console is the new version and they are running on the same URL and port as before.

## Uninstall a Kubernetes release of Operational Decision Manager

To uninstall and delete a template along with all of the associated releases, use the following command:

```console
$ kubectl delete -f generated-k8s-templates.yaml
```

> **Note**: The command removes all the Kubernetes components associated with the chart, even Persistent Volume Claims (PVCs), which might contain valuable data.