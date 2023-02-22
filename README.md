# mysqld

Table of contents
Prerequisites:
Secret
Services
StatefulSet
Verifying the replica set deployment and accessing the replica set
Create Sample Tables and Data:
Conclusion:
Reading Time: 4 minutes

This blog aims to explain each of the components required to deploy MySQL statefulset cluster on Kubernetes.

While deploying the MySQL on Kubernetes, what object type should be used and why? Deployments or StatefulSets?

The answer is StatefulSet. Let’s discuss!

StatefulSet is the Kubernetes object used to manage stateful applications.It is preferred over deployments as it provides guarantees about the ordering and uniqueness of these Pods i.e. the management of volumes is better with stateful sets.

Why do We Need MySQL Statefulset?


MySQL is going to be a stateful application i.e. it stores data (like tables, users) inside a volume. If the data is stored in pod ephemeral storage, then the data will get erased once the pod restarts.

Also, MySQL may have to be scaled to more than one pod in caseload increases.

All these operations have to be done in such a way that data consistency is maintained across all the pods like mysql-0, mysql-1.

Prerequisites:
A Kubernetes Cluster (I will be using minikube single node cluster on my local system)
Secret
Secrets in Kubernetes are the objects used for supplying sensitive information to containers.They are like ConfigMaps with the difference that data is store in a base 64 encoded format.

For the security of our MySQL cluster, it is wise to restrict access to the database with a password. We will use Secrets to mount our desired passwords to the containers.

In this tutorial, we use base64 encoded to store ‘MYSQL_ROOT_PASSWORD’. For example:

$ echo -n "password" | base64
cGFzc3dvcmQ==
Keeping note of the outputted value and create a new file named secret.yml

$ vi secret.yml
apiVersion: v1
kind: Secret
metadata: 
    name: mysecret
type: Opaque
data:
   ROOT_PASSWORD: cGFzc3dvcmQ=
Now run the kubectl apply command to create the secret in Kubernetes.

$ kubectl apply -f secret.yml 
secret/mysecret created
We can use the kubectl describe secret command to display additional information about the resource.

$ kubectl describe secret/mysecret
Name:         mysecret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
ROOT_PASSWORD:  8 bytes
Under data we can see the key we defined in the secret.yml file, however, we do not see the actual value. We only know that the value is 8 bytes in length. We wouldn’t expect to see the value as wouldn’t be very secure if we could.
