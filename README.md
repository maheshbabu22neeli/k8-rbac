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

- Create a Role with the following permissions:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: roboshop
  name: trainee
rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
```
- 
