# cert-manager-example

We use an off-the-shelf component [cert-manager](https://github.com/jetstack/cert-manager) as an example to demonstrate
how [Config Sync](https://cloud.google.com/kubernetes-engine/docs/add-on/config-sync) syncs an unstructured repo to your cluster using Helm template.

To sync the Helm chart to your cluster, you can run `helm template` and commit the rendered manifest to your repo. 

## Prerequisite
- Install [Helm](https://helm.sh/) to render the manifests.
- Install [Config Sync](https://cloud.google.com/kubernetes-engine/docs/add-on/config-sync/how-to/installing) to sync the resources to your cluster.

## Steps
1. Fork or check out this repository.
2. Render the manifests: 
    - Render from the remote chart:
        `helm template my-cert-manager jetstack/cert-manager --version 1.3.0 --namespace cert-manager --output-dir manifests`.
    - Render from your local chart:
        `helm template my-cert-manager cert-manager --namespace cert-manager --output-dir manifests`.
3. Check in the rendered manifests and commit to your repo.
4. Configure your ConfigManagement object or RootSync object as below:
    ```
    # config-management.yaml
    
    apiVersion: configmanagement.gke.io/v1
    kind: ConfigManagement
    metadata:
      name: config-management
    spec:
      # clusterName is required and must be unique among all managed clusters
      clusterName: my-cluster
      git:
        syncRepo: https://github.com/nan-yu/cert-manager-example
        syncBranch: main
        secretType: ssh
        policyDir: manifests
      sourceFormat: unstructured
    ```
    
    OR
    ```
    # root-sync.yaml
    # If you are using a Config Sync version earlier than 1.7,
    # use: apiVersion: configsync.gke.io/v1alpha1
    apiVersion: configsync.gke.io/v1beta1
    kind: RootSync
    metadata:
      name: root-sync
      namespace: config-management-system
    spec:
      git:
        auth: ssh
        branch: main
        dir: manifests
        repo: https://github.com/nan-yu/cert-manager-example
        secretRef:
          name: ssh-key
      sourceFormat: unstructured
    ```
5. Verify the resources are successfully synced to your cluster.

# Uninstall the cert-manager chart
1. Remove the rendered manifests from the repo: `git rm -rf manifests/cert-manager && git commit -m "uninstall cert-manager" && git push origin main`
2. Wait until RootSync is synced successfully.
3. Remove the `cert-manager` namespace from the repo: `git rm manifests/namespace-cert-manager.yaml && git commit -m "remove the cert-manager namespace" && git push origin main`
4. Wait until RootSync is synced successfully.
