# POC [Crossplane](https://crossplane.io/docs/v1.4/)


Challenge: 2021 is the year of IaC. We've built a cloud management platform (see the API spec here) to host a service catalog. Our customer, however, should not directly call its API but consume our services as infrastructure-as-code.
 
Goal: Please setup a crossplane IaC lab and demonstrate how a code change triggers the provisioning/deletion of a resource in the Cloud. You may exclude other Day2 operations from the demo.
 
Guiding question and answers: 

1) How does crossplane compare to k8s operators and terraform?
    - It adheres to the principles of Infrastructure as Data. It’s Kubernetes-native, so you use Kubernetes Custom Resources (YAML — i.e. text) to declaratively provision Cloud infrastructure.
    - No state file. Unlike Terraform and Pulumi, Crossplane doesn’t use a state file. That said, as a Kubernetes Operator, it has a controller to reconcile desired state against current state, making use of etcd to do so. 
    - Furthermore, unlike Terraform and Pulumi, if someone tries to do something sneaky like update a Cloud resource using the CLI or an admin console, Crossplane won’t let you get away with it. It is the source of truth.

2) How does crossplane integrate with existing IaC tools like ansible or terraform?
    - Terraform
        Terraform, by Hashicorp, is very popular platform-agnostic tool for provisioning infrastructure for various Public Cloud and Private Cloud frameworks. It uses its own proprietary JSON-esque language, Hashicorp Configuration Language (HCL) for defining your infrastructure. It supports some very crude loops and conditionals, as well as variables. Terraform relies on a JSON state file to keep track of the infrastructure that it created. Kind of a structured log.
    - Ansible
        While it may have started out as a configuration management tool, Ansible can now also be used to provision and manage Cloud infrastructure. Unlike Terraform and Pulumi, Ansible uses YAML to define infrastructure. Also unlike Terraform and Pulumi, Ansible is stateless. 
    - So, we cannot really compare them, as there approach is different, Ansible can be used for the Configuration Management part after the infrastructure is provisioned using Crossplane or Terraform.
    - We can run shell scripts from Kubernetes Pods to do the configuration Management part using ansible, can be a use case of integration of Crossplane with Ansible.
    - Likewise Terraform scripts can be run from the infra created using Crossplane. 

3) How would we integrate existing provisioning APIs like VRA or AWS API but also other self-written APIs into crossplane? Which requirements do we have for an API to be integrated with crossplane?
    - crossplane providers is what we need to talk to the various APIs. In order to provision a resource, a Custom Resource Definition (CRD) needs to be registered in your Kubernetes cluster and its controller should be watching the Custom Resources those CRDs define. Provider packages contain many Custom Resource Definitions and their controllers. These providers will accept the authorization credentails to authenticate and provision infrastructure. Essential is to refer to the Provider config while provissioning resources.

4) How would you setup a production crossplane environment to manage 10k workloads (i.e., infra services)? (e.g., how many K8s clusters for crossplane? which other constraints do you see?).
    - Here preference will be ofcourse a CI/CD tool, eg. GitlabCI or some other GitOps tool that will talk to the Provisioner Kubernetes Cluster, Environments/Projects seperated by Kubernetes Namespaces with suitable configuration. 
    - 10k workload of infra services, may be in different region, different Cloud provider Accounts / Projects has to be properly maintained in SCM and defined pipeline to deploy changes.  

[API Documentation](https://crossplane.io/docs/v1.4/api-docs/overview.html) 

# LAB Setup.

# Pre-requisite for self hosted Crossplane
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)/[Kind](https://github.com/kubernetes-sigs/kind) as Local kubernetes cluster
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [helm](https://helm.sh/)
- [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/install-macos.html)
- [gcloud cli](https://cloud.google.com/sdk/docs/quickstart)
- [crossplane cli](https://crossplane.io/docs/v1.4/getting-started/install-configure.html)
- [crossplane installation on minikube](https://crossplane.io/docs/v1.4/getting-started/install-configure.html)
```
$ kubectl create namespace crossplane-system
$ helm repo add crossplane-stable https://charts.crossplane.io/stable
$ helm repo update
$ helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```

* Note: in this example crossplane was installed in Minikube.

# What we deployed

```
% kubens  
crossplane-system
default
kube-node-lease
kube-public
kube-system

% kubens crossplane-system
Context "minikube" modified.
Active namespace is "crossplane-system".

% kubectl get all         
NAME                                          READY   STATUS    RESTARTS   AGE
pod/crossplane-6f974db97-7ddsh                1/1     Running   0          28s
pod/crossplane-rbac-manager-dd8d65f77-csq2j   1/1     Running   0          28s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/crossplane                1/1     1            1           28s
deployment.apps/crossplane-rbac-manager   1/1     1            1           28s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/crossplane-6f974db97                1         1         1       28s
replicaset.apps/crossplane-rbac-manager-dd8d65f77   1         1         1       28s
```
# Deploy example resources in Public Cloud Platforms

## Pre-requisites
- A Kubernetes cluster ( Can be either On-Prem, AKS, EKS, GKE, Kind ). Minikube in this case was used.
- You have an existing Google Cloud project.
- You’ve created a Service Account in Google Cloud and SDK configured in workmachine.
- You’ve created a Google Kubernetes Engine (GKE) cluster before.
- An AWS account with IAM users credentails and configured CLI.

## Wiki for deploying resources to public cloud plaforms and deployed resources can be found below:

- [AWS](./src/aws/README.md)
- [GCP](./src/gcp/README.md)

# Conclusion

- Crossplane is a Kubernetes-native tool for provisioning Cloud infrastructure.
- Unlike Terraform and Pulumi, which keep state files, and let us alter Cloud infrastructure outside the tool (with possibly devastating consequences), Crossplane doesn’t let get away with that( kind of Chef/puppet idempotent nature).
- Like Ansible, it adheres to the principles of Infrastructure as Data, using YAML to declaratively provision Cloud resources. 
- Definitely some more Providers expeceted or we need to write our own providers to work with, open source allows that and expected too.
- Documentaion is what I feel must be more descriptive, hope will extend with time.
- Kubernetes Cluster necessary to deploy resources, CI/CD system also can be incorporated to make further automated commission/decommission of infra. 
