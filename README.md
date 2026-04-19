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

### Role and Role Binding
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
    - rolearn: arn:aws:iam::204427113986:role/eksctl-roboshop-nodegroup-spot-NodeInstanceRole-k1FHX0MGiC44
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

