## Create Role

We must create a role to allow EKS to manage aws resources such as EC2, load balancer, etc.

```javascript
# ./assume-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "eks.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

Let's do this via the cli and let's keep a reference to the role's ARN (Amazon Resource Name) as we will need it when we create our cluster.

```bash
# Create role
role_arn=`aws iam create-role --role-name eks-role --assume-role-policy-document file://assume-policy.json | jq .Role.Arn | sed s/\"//g`

# Attach policy
aws iam attach-role-policy --role-name eks-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
```

Next we create the VPC for the cluster. We will use a template file which can be downloaded via curl.

`curl https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-05-08/amazon-eks-vpc-sample.yaml -o vpc.yaml`

Once downloaded, we create the VPC refencing `vpc.yaml`

```bash
aws cloudformation deploy --template-file vpc.yaml --stack-name eks-vpc
```

Next we want save our stack details to access the subnets on which we want to drop our EKS nodes into.

```bash
aws cloudformation list-stack-resources --stack-name eks-vpc > stack.json
```

Finally, we may go on to create our cluster. Note that under `--resources-vpc-config` we're referencing the subnet ids that we put into `stack.json` when creating our VPC stack.

```bash
aws eks create-cluster \
--name eks-cluster \
--role-arn $role_arn \
--resources-vpc-config subnetIds=subnet-08864bcdcda48bada,subnet-07cb81d15326fa5ba,subnet-0d6e64ceabab76e60,securityGroupIds=sg-0e69b4c370e590564,endpointPublicAccess=true,endpointPrivateAccess=false
```

Done! Now we run a few commands and make sure everything is working.

```bash
aws eks list-clusters
aws eks describe-cluster --name eks-cluster
```

Finally, it'd be a good idea to wait for the cluster to be "Active" rather than "Creating". It usually takes about 5 minutes. You may check the status in the aws consoles or by running `aws eks describe-cluster --name eks-cluster`.

[Next](https://https://github.com/Jonroslu/KnowledgeBase/blob/master/awk-eks-setup/3-access-cluster.md)