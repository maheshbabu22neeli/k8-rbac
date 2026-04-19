# k8-rbac
Kubernetes Role-Based Access Control


## RBAC
- Kubernetes  is a platform as a service software - PaaS. And it has its own authentication and authorization mechanism.
- RBAC is a method of regulating access to computer or network resources based on the roles of individual users within an enterprise.
- In Kubernetes, RBAC is used to control access to the Kubernetes API and resources.
- RBAC allows you to define roles and permissions for users and groups, and then assign those roles to users or groups.
- This way, you can control who has access to what resources in your Kubernetes cluster.
- RBAC is implemented using Kubernetes API objects such as Role, ClusterRole, RoleBinding, and ClusterRoleBinding.
- Role: A Role defines a set of permissions within a namespace. It can be used to grant access to resources within a specific namespace.
- ClusterRole: A ClusterRole defines a set of permissions that can be applied across the entire cluster. It can be used to grant access to resources across all namespaces.
- RoleBinding: A RoleBinding grants the permissions defined in a Role to a user or group within a specific namespace.
- ClusterRoleBinding: A ClusterRoleBinding grants the permissions defined in a ClusterRole to a user or group across the entire cluster.
- RBAC is an important aspect of Kubernetes security, as it allows you to control access to your cluster and resources, and helps to prevent unauthorized access and potential security breaches.
- RBAC is a powerful tool for managing access to your Kubernetes cluster, and it is essential for maintaining the security and integrity of your cluster.
- RBAC is a critical component of Kubernetes security, and it is important to understand how to use it effectively to protect your cluster and resources.

### Role and Role Binding     -- Trainee 
- Role and Role-Binding are namespace scoped. So, we will create Role and Role Binding in `roboshop` namespace.
- Create policy with `ClusterDescribe` permissions in AWS console.
- Create User from AWS console and assign the above created policy to the user.

```shell
- Create namespace using `kubectl create namespace roboshop`
$ kubectl apply -f 01-role.yaml
$ kubectl get role -n roboshop
NAME           CREATED AT
trainee-role   2026-04-19T04:11:13Z

$ kubectl apply -f 02-role-binding.yaml
$ kubectl get rolebinding -n roboshop
NAME              ROLE                AGE
trainee-binding   Role/trainee-role   10s
```
- Now, create aws-auth configmap in kube-system namespace and add the user ARN to the mapRoles section.
- This can be done by following command:
```shell
$ kubectl get configmap aws-auth -n kube-system -o  yaml                              -> get existing configmap
apiVersion: v1
data:
  mapRoles: |
    - rolearn: arn:aws:iam::<ACCOUNT-ID>:role/eksctl-roboshop-nodegroup-spot-NodeInstanceRole-k1FHX0MGiC44
      groups:
      - system:bootstrappers
      - system:nodes
      username: system:node:{{EC2PrivateDNSName}}
kind: ConfigMap
metadata:
  creationTimestamp: "2026-04-19T03:15:37Z"
  name: aws-auth
  namespace: kube-system
  resourceVersion: "1252"
  uid: fa9bc09e-bd60-4c84-baf4-54ff8bd9cce1

```
- Now add mapUsers section for the user ARN in the above configmap and apply the changes.
```shell
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: trainee-binding
  namespace: roboshop

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: trainee-role

subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: ramesh # "name" is case sensitive, and it is created through aws console
    
$ kubectl apply -f 03-aws-auth.yaml
Warning: resource configmaps/aws-auth is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
configmap/aws-auth configured
```
- Now, ask ramesh to login and verify the access to the cluster using kubectl command.
```shell
Create an Ec2 instance 
Install kubectl
Do aws configure 
$ aws eks update-kubeconfig --name roboshop --region us-east-1
Added new context arn:aws:eks:us-east-1:<ACCOUNT-ID>:cluster/roboshop to /home/ec2-user/.kube/config

$ kubectl get pods                                      -> ramesh does not have access to default namespace
Error from server (Forbidden): pods is forbidden: User "ramesh" cannot list resource "pods" in API group "" in the namespace "default"

13.222.134.210 | 172.31.16.221 | t3.micro | null
[ ec2-user@ip-172-31-16-221 ~ ]$ kubectl get pods -n roboshop   -> ramesh have access to roboshop namespace
No resources found in roboshop namespace.

$ kubectl get deployment -n roboshop                    -> ramesh does not have access to deployments in roboshop namespace
Error from server (Forbidden): deployments.apps is forbidden: User "ramesh" cannot list resource "deployments" in API group "apps" in the namespace "roboshop"
```

### Role and Role Binding     -- Admin
- Can use the same policy with `ClusterDescribe` permissions in AWS console.
- Create User (suresh) from AWS console and assign the above created policy to the user.

```shell
$ kubectl apply -f 04-admin-role.yaml
$ kubectl get role -n roboshop
$ kubectl apply -f 05-admin-role-binding.yaml
$ kubectl get rolebinding -n roboshop

```
- Now, create aws-auth configmap in kube-system namespace and add the user ARN to the mapRoles section.
- Now add mapUsers section for the user ARN in the above configmap and apply the changes.
```shell
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: "2026-04-19T03:15:37Z"
  name: aws-auth
  namespace: kube-system

data:
  mapRoles: |
    - rolearn: arn:aws:iam::<ACCOUNT-ID>:role/eksctl-roboshop-nodegroup-spot-NodeInstanceRole-k1FHX0MGiC44
      groups:
      - system:bootstrappers
      - system:nodes
      username: system:node:{{EC2PrivateDNSName}}
  mapUsers: |
    - userarn: arn:aws:iam::<ACCOUNT-ID>:user/ramesh
      groups:
      - trainee-binding
      username: ramesh
    - userarn: arn:aws:iam::<ACCOUNT-ID>:user/suresh
      groups:
      - admin-binding
      username: suresh
         
$ kubectl apply -f 03-aws-auth.yaml
Warning: resource configmaps/aws-auth is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
configmap/aws-auth configured
```
- Now, ask suresh to login and verify the access to the cluster using kubectl command.
```shell
Create an Ec2 instance 
Install kubectl
Do aws configure 

$ aws sts get-caller-identity
{
    "UserId": "AIDAS7GGOGYBHEXOOYDVS",
    "Account": "<ACCOUNT-ID>",
    "Arn": "arn:aws:iam::<ACCOUNT-ID>:user/suresh"
}

$ aws eks update-kubeconfig --name roboshop --region us-east-1
Updated context arn:aws:eks:us-east-1:204427113986:cluster/roboshop in /home/ec2-user/.kube/config

$ kubectl get pods -n roboshop                         -> suresh has access to pod
No resources found in roboshop namespace.

$ kubectl get deployments -n roboshop                  -> suresh has access to deployments
No resources found in roboshop namespace.

$ kubectl get pv                                        -> suresh does not have access to peristent volumes at cluster scope
Error from server (Forbidden): persistentvolumes is forbidden: User "suresh" cannot list resource "persistentvolumes" in API group "" at the cluster scope

```


### Cluster-Role and Cluster Role Binding
- Cluster-Role and Cluster Role Binding are cluster scoped.
- Create Cluster Role with `ClusterDescribe` permissions in AWS console.