ubuntu@ip-172-31-16-229:~/EKS$ aws eks update-kubeconfig --region us-east-2 --name devopsshack-cluster

An error occurred (ResourceNotFoundException) when calling the DescribeCluster operation: No cluster found for name: devopsshack-cluster.
ubuntu@ip-172-31-16-229:~/EKS$ aws eks update-kubeconfig --region us-east-1 --name devopsshack-cluster
Added new context arn:aws:eks:us-east-1:160885251727:cluster/devopsshack-cluster to /home/ubuntu/.kube/config
ubuntu@ip-172-31-16-229:~/EKS$ eksctl cluste-info
eksctl: command not found
ubuntu@ip-172-31-16-229:~/EKS$ kubectl cluster-info
Kubernetes control plane is running at https://1E3C784699B7E42CE79EC97A01A4755F.gr7.us-east-1.eks.amazonaws.com
CoreDNS is running at https://1E3C784699B7E42CE79EC97A01A4755F.gr7.us-east-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
ubuntu@ip-172-31-16-229:~/EKS$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   16m
ubuntu@ip-172-31-16-229:~/EKS$
ubuntu@ip-172-31-16-229:~/EKS$
ubuntu@ip-172-31-16-229:~/EKS$ kubectl get nodes
NAME                         STATUS   ROLES    AGE   VERSION
ip-10-0-0-71.ec2.internal    Ready    <none>   82m   v1.31.3-eks-59bf375
ip-10-0-1-135.ec2.internal   Ready    <none>   83m   v1.31.3-eks-59bf375
ip-10-0-1-236.ec2.internal   Ready    <none>   83m   v1.31.3-eks-59bf375
ubuntu@ip-172-31-16-229:~/EKS$ kubectl create ns webapps
namespace/webapps created
ubuntu@ip-172-31-16-229:~/EKS$ kubectl get ns
NAME              STATUS   AGE
default           Active   102m
kube-node-lease   Active   102m
kube-public       Active   102m
kube-system       Active   102m
webapps           Active   6s
ubuntu@ip-172-31-16-229:~/EKS$
ubuntu@ip-172-31-16-229:~/EKS$
ubuntu@ip-172-31-16-229:~/EKS$ sudo vi svc.yml
ubuntu@ip-172-31-16-229:~/EKS$ kubectl applt -f svc.yml
error: unknown command "applt" for "kubectl"

Did you mean this?
        apply
ubuntu@ip-172-31-16-229:~/EKS$ kubectl apply -f svc.yml
serviceaccount/jenkins created
ubuntu@ip-172-31-16-229:~/EKS$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   103m
ubuntu@ip-172-31-16-229:~/EKS$
ubuntu@ip-172-31-16-229:~/EKS$
ubuntu@ip-172-31-16-229:~/EKS$ sudo vi role.yml
ubuntu@ip-172-31-16-229:~/EKS$
ubuntu@ip-172-31-16-229:~/EKS$
ubuntu@ip-172-31-16-229:~/EKS$ kubectl apply -f role.yml
role.rbac.authorization.k8s.io/app-role created
ubuntu@ip-172-31-16-229:~/EKS$ sudo vi bind.yml
ubuntu@ip-172-31-16-229:~/EKS$ kubectl apply -f bind.yml
rolebinding.rbac.authorization.k8s.io/app-rolebinding created
ubuntu@ip-172-31-16-229:~/EKS$
ubuntu@ip-172-31-16-229:~/EKS$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   105m
ubuntu@ip-172-31-16-229:~/EKS$
ubuntu@ip-172-31-16-229:~/EKS$
ubuntu@ip-172-31-16-229:~/EKS$ sudo vi secret.yml
ubuntu@ip-172-31-16-229:~/EKS$ sudo vi secret.yml
ubuntu@ip-172-31-16-229:~/EKS$ kubectl apply -f secret.yml -n webapps
secret/secname created
ubuntu@ip-172-31-16-229:~/EKS$ kubectl describe secret secname -n webapps
Name:         secname
Namespace:    webapps
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: jenkins
              kubernetes.io/service-account.uid: c943fb4a-a077-4904-b116-7c9100694ae4

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1107 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkFOYmNpQW4yLXNtdERXNVk0MGxzQnZYUm1LRU9meGhvTElYbXZpQ2s3d28ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ3ZWJhcHBzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InNlY25hbWUiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiamVua2lucyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImM5NDNmYjRhLWEwNzctNDkwNC1iMTE2LTdjOTEwMDY5NGFlNCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDp3ZWJhcHBzOmplbmtpbnMifQ.IIEVwcniNWJbgJAkBYsMVcgoNT1UIjT54XynTkux2ZhnQk6DMtFyXYf6n0mU7un1mko7N-5dyqllivc5Gio3WoDjlRApnqcXJFSQeSKNHZYpp9o4TDifuZ7JvmYT7yDhCXUHfclhyMDOQgajE_vg9gkOq9kKy9u-jK9pX3Ejdxrz0UBZClqwsvf_lMW1sHuPRnHeZwZfYcm2f-cIrxrMTUUr3CLH96D7ynQbUZwq1dY2E3KmfA7ByezeP3p-mlmKmDR-rSBpH_BHOAf-QXfdJ2Gn5QnKD8a5jvZ3JRrWpKSOz_4RIjKJGaXVycvuBcmSi7n5u8iHwaKPQsDTMg9tbQ
