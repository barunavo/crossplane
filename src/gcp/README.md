# Steps to deploy example resources


- install gcp provider
    ```
    % cd src/gcp
    % kubectl crossplane install provider crossplane/provider-gcp:v0.15.0
    provider.pkg.crossplane.io/crossplane-provider-gcp created
    
    % kubectl get providers
    NAME                      INSTALLED   HEALTHY   PACKAGE                           AGE
    crossplane-provider-gcp   True        True      crossplane/provider-gcp:v0.15.0   20s

- install secret for GCP API authentication keys
    ```
    % kubectl apply -f secret.yml
    ```

- install provider config that allows to authenticate to GCP APIs
    ```
    % kubectl apply -f provider-config.yml
    ```

- gke cluster
    ```
    % kubectl apply -f ./gke.yml
    gkecluster.container.gcp.crossplane.io/devops-cluster unchanged
    nodepool.container.gcp.crossplane.io/devops-cluster configured
    ```

- connect to GKE cluster
    ```
    gcloud container clusters get-credentials devops-cluster --region us-east1 --project <project-name>
    ```

- cleanup
    ```
    % kubectl delete -f ./gke.yml
    ```
