# cert-manager-example

We use an off-the-shelf component [cert-manager](https://github.com/jetstack/cert-manager) as an example to demonstrate
how [Config Sync](https://cloud.google.com/kubernetes-engine/docs/add-on/config-sync) syncs an unstructured repo to your cluster using Helm template.

To sync the Helm chart to your cluster, you can run `helm template` and commit the rendered manifest to your repo. 