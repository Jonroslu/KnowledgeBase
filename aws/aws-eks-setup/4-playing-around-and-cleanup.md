## Playing with our cluster

There are some resources under the `k8s` folder which we can use to play around with our cluster.

```bash
cd ./k8s

kubectl create ns test-app

# lets create some resources.
kubectl apply -n test-app -f secrets/secret.yaml
kubectl apply -n test-app -f configmaps/configmap.yaml
kubectl apply -n test-app -f deployments/deployment.yaml

# remember to change the `type: LoadBalancer`
kubectl apply -n test-app -f services/service.yaml
```

You should now see all the pods.

```bash
$ kubectl get pods -A    
NAMESPACE        NAME                             READY   STATUS    RESTARTS   AGE
kube-system      aws-node-lvzq7                   1/1     Running   0          16m
kube-system      coredns-55c5fcd78f-8tlhf         1/1     Running   0          157m
kube-system      coredns-55c5fcd78f-xkcwg         1/1     Running   0          157m
kube-system      kube-proxy-d5856                 1/1     Running   0          16m
test-namespace   example-deploy-99754ccb9-bgf6z   1/1     Running   0          105s
test-namespace   example-deploy-99754ccb9-jpkz7   1/1     Running   0          105s
```

The `service.yaml` file tells kubernetes that an elastic load balancer `type: LoadBalancer` has to be created and attached to our pods.

Run the following to get the public IP.

```bash
$ kubectl get svc -n test-namespace
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP                                                               PORT(S)        AGE
example-service   LoadBalancer   10.100.68.158   adc4b8e90b875452dac27d41cab322e0-1832908039.us-east-1.elb.amazonaws.com   80:30107/TCP   4m53s
```

You may use this external IP to access the service.

## Cleanup

```bash
eksctl delete cluster --name getting-started-eks-1

aws eks delete-nodegroup --cluster-name getting-started-eks --nodegroup-name test
aws eks delete-cluster --name getting-started-eks

aws iam detach-role-policy --role-name getting-started-eks-role --policy-arn  arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
aws iam delete-role --role-name getting-started-eks-role

aws iam detach-role-policy --role-name getting-started-eks-role-nodes --policy-arn  arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
aws iam detach-role-policy --role-name getting-started-eks-role-nodes --policy-arn  arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
aws iam detach-role-policy --role-name getting-started-eks-role-nodes --policy-arn  arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly

aws iam delete-role --role-name getting-started-eks-role-nodes

aws cloudformation delete-stack --stack-name getting-started-eks
```