# Steps to deploy example AWS resources

- install provider
    ```
    % kubectl apply -f aws-provider.yml
    ````
- install k8s secret with AWS IAM user keys for authentication
    ```
    % AWS_PROFILE=default && echo -e "[default]\naws_access_key_id = $(aws configure get aws_access_key_id --profile $AWS_PROFILE)\naws_secret_access_key = $(aws configure get aws_secret_access_key --profile $AWS_PROFILE)" > creds.conf
    % kubectl create secret generic aws-secret-creds -n crossplane-system --from-file=creds=./creds.conf
    % kubectl apply -f provider-config.yaml 
    ```
- install AWS VPC and related resources
    ```
    % cd src/aws
    % kubectl apply -f vpc/ 
    ```
- install a RDS instance in the deployed VPC
    ```
    % kubectl apply -f rds/
    ```
- cleanup
    ```
    % kubectl delete -f rds/
    % kubectl delete -f vpc/
    ```



