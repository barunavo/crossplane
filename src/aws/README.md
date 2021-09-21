# Steps to deploy example AWS resources

- install provider
    ```
    % kubectl apply -f aws-provider.yml
    ````
- install k8s secret with AWS IAM user keys for authentication
    ```
    % AWS_PROFILE=default && echo -e "[default]\naws_access_key_id = $(aws configure get aws_access_key_id --profile $AWS_PROFILE)\naws_secret_access_key = $(aws configure get aws_secret_access_key --profile $AWS_PROFILE)" > creds.conf
    % kubectl create secret generic aws-secret-creds -n crossplane-system --from-file=creds=./credentails.conf
    % kubectl apply -f provider-config.yaml 
    ```
- install AWS VPC and related resources
    ```
    % cd src/aws
    % kubectl apply -f vpc/ 
    vpc.ec2.aws.crossplane.io/production-vpc created
    subnet.ec2.aws.crossplane.io/prod-subnet-1 created
    subnet.ec2.aws.crossplane.io/prod-subnet-2 created
    subnet.ec2.aws.crossplane.io/prod-subnet-3 created
    internetgateway.ec2.aws.crossplane.io/production-internetgateway created
    routetable.ec2.aws.crossplane.io/production-routetable created
    ```
- install a RDS instance in the deployed VPC
    ```
    % kubectl apply -f rds/
    dbsubnetgroup.database.aws.crossplane.io/prod-subnet-group created
    rdsinstance.database.aws.crossplane.io/production-rds created
    securitygroup.ec2.aws.crossplane.io/rds-access-sg created
    ```
- cleanup
    ```
    % kubectl delete -f rds/,
    dbsubnetgroup.database.aws.crossplane.io "prod-subnet-group" deleted
    rdsinstance.database.aws.crossplane.io "production-rds" deleted
    securitygroup.ec2.aws.crossplane.io "rds-access-sg" deleted
    % kubectl delete -f vpc/
    vpc.ec2.aws.crossplane.io "production-vpc" deleted
    subnet.ec2.aws.crossplane.io "prod-subnet-1" deleted
    subnet.ec2.aws.crossplane.io "prod-subnet-2" deleted
    subnet.ec2.aws.crossplane.io "prod-subnet-3" deleted
    internetgateway.ec2.aws.crossplane.io "production-internetgateway" deleted
    routetable.ec2.aws.crossplane.io "production-routetable" deleted
    ```

