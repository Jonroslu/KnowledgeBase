## Access Cluster

Now that our cluster is `active`, let's go ahead and access it. Before doing so, however, we need a kubeconfig, so let's take care of that first.

```bash
aws eks update-kubeconfig \
--name eks-cluster \
--region us-east-1
```

This will download the kubeconfig and place it in the directory `~/.kube/config`. We will use this to access the cluster with `kubectl`.

Let's bring the config over to our working directory.

```bash
cp ~/.kube/config .
```

Now we can try to run kubectl.

```bash
kubectl get nodes
```

And you will notice that it responds with `No resources found in default namespace.`.

This is because we have only created the control plane, or the master. We get `No resources found` because the Kubernetes controller cannot find any resources. Let's fix this by adding some nodes.

We will create a NodeGroup with `small` ec2 instances.

First, we create the role for our NodeGroup.

```javascript
# ./assume-node-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

```bash
role_arn=`aws iam create-role --role-name eks-nodes-role --assume-role-policy-document file://assume-node-policy.json | jq .Role.Arn | sed s/\"//g`
```

Now let's add the policies that we need our NodeGroup role to have.

```bash
# NodeGroup security
aws iam attach-role-policy --role-name eks-nodes-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy

# Allow container networking on NodeGroup
aws iam attach-role-policy --role-name eks-nodes-role --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy

# For container registry so GroupNode can pull from there
aws iam attach-role-policy --role-name eks-nodes-role --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
```

Now that we have our role ready, let's create our NodeGroup.

```bash
# Create NodeGroup referencing our cluster
aws eks create-nodegroup \
--cluster-name eks-cluster \
--nodegroup-name eks-test \
--node-role $role_arn \
--subnets subnet-08864bcdcda48bada \
--disk-size 200 \
--scaling-config minSize=1,maxSize=2,desiredSize=1 \
--instance-types t2.small
```

It will take a few until it finished, but once it is `CREATED` we will be able to connect with kubectl.

[Next](https://https://github.com/Jonroslu/KnowledgeBase/blob/master/aws/awk-eks-setup/4-playing-around-and-cleanup.md)