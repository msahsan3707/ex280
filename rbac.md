<pre>
<h1> RBAC </h1>

[mahsan@r9 ex280]$ oc get clusterrolebinding
NAME                                                                        ROLE                                                                                    AGE
basic-users                                                                 ClusterRole/basic-user                                                                  93d
cluster-admin                                                               ClusterRole/cluster-admin                                                               93d
cluster-admin-0                                                             ClusterRole/cluster-admin                                                               3h45m
cluster-admins                                                              ClusterRole/cluster-admin                                                               93d
cluster-autoscaler                                                          ClusterRole/cluster-autoscaler                                                          93d
cluster-autoscaler-operator                                                 ClusterRole/cluster-autoscaler-operator                                                 93d
cluster-monitoring-operator                                                 ClusterRole/cluster-monitoring-operator                                                 93d
cluster-network-operator                                                    ClusterRole/cluster-admin                                                               93d
cluster-readers                                                             ClusterRole/cluster-reader                                                              93d
cluster-samples-operator                                                    ClusterRole/cluster-samples-operator                                                    93d
cluster-samples-operator-imageconfig-reader                                 ClusterRole/cluster-samples-operator-imageconfig-reader                                 93d
cluster-samples-operator-proxy-reader                                       ClusterRole/cluster-samples-operator-proxy-reader                                       93d
cluster-status-binding                                                      ClusterRole/cluster-status                   

[mahsan@r9 ex280]$ oc describe clusterrolebindings self-provisioners
Name:         self-provisioners
Labels:       <none>
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  self-provisioner
Subjects:
  Kind   Name                        Namespace
  ----   ----                        ---------
 <b> Group  system:authenticated:oauth </b>

oc edit clusterrolebindings self-provisioners

  
  # Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
 <b>   rbac.authorization.kubernetes.io/autoupdate: "false" </b>
  creationTimestamp: "2024-12-03T08:13:44Z"
  name: self-provisioners
  resourceVersion: "103530"
  uid: 1a36a74a-94b9-450c-9121-c5c13eb18b46
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: self-provisioner
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:authenticated:oauth

 
[mahsan@r9 ex280]$ oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
clusterrole.rbac.authorization.k8s.io/self-provisioner removed: "system:authenticated:oauth"

[mahsan@r9 ex280]$ oc policy add-role-to-user admin admin
clusterrole.rbac.authorization.k8s.io/admin added: "admin"
[mahsan@r9 ex280]$ oc policy add-role-to-user admin developer
clusterrole.rbac.authorization.k8s.io/admin added: "developer"

[mahsan@r9 ex280]$ oc get rolebindings.rbac
NAME                               ROLE                                           AGE
admin                              ClusterRole/admin                              102s
admin-0                            ClusterRole/admin                              92s
machine-config-controller-events   ClusterRole/machine-config-controller-events   93d
machine-config-daemon-events       ClusterRole/machine-config-daemon-events       93d
machine-os-builder-events          ClusterRole/machine-os-builder-events          93d
system:deployers                   ClusterRole/system:deployer                    93d
system:image-builders              ClusterRole/system:image-builder               93d
system:image-pullers               ClusterRole/system:image-puller                93d
[mahsan@r9 ex280]$


[mahsan@r9 tmp]$ oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
clusterrole.rbac.authorization.k8s.io/self-provisioner removed: "system:authenticated:oauth"

[mahsan@r9 tmp]$ oc policy add-role-to-user admin mahsan
clusterrole.rbac.authorization.k8s.io/admin added: "mahsan"
[mahsan@r9 tmp]$

[mahsan@r9 tmp]$ oc get rolebindings.rbac
NAME                               ROLE                                           AGE
admin                              ClusterRole/admin                              40s
machine-config-controller-events   ClusterRole/machine-config-controller-events   95d
machine-config-daemon-events       ClusterRole/machine-config-daemon-events       95d
machine-os-builder-events          ClusterRole/machine-os-builder-events          95d
mahsan                             ClusterRole/mahsan                             61s
system:deployers                   ClusterRole/system:deployer                    95d
system:image-builders              ClusterRole/system:image-builder               95d
system:image-pullers               ClusterRole/system:image-puller                95d
[mahsan@r9 tmp]$

[mahsan@r9 tmp]$ oc adm groups new dev-group
group.user.openshift.io/dev-group created
[mahsan@r9 tmp]$ oc adm groups new da-group
group.user.openshift.io/da-group created
[mahsan@r9 tmp]$ oc get groups
NAME        USERS
da-group
dev-group

[mahsan@r9 tmp]$ oc adm groups add-users dev-group developer
group.user.openshift.io/dev-group added: "developer"
[mahsan@r9 tmp]$

[mahsan@r9 tmp]$ oc adm groups add-users da-group natasha
group.user.openshift.io/da-group added: "natasha"
[mahsan@r9 tmp]$ oc get groups
NAME        USERS
da-group    natasha
dev-group   developer
[mahsan@r9 tmp]$

[mahsan@r9 tmp]$ oc new-project auth-rbac
Now using project "auth-rbac" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

[mahsan@r9 tmp]

[mahsan@r9 tmp]$ oc new-project auth-rbac
Now using project "auth-rbac" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

[mahsan@r9 tmp]$ oc policy add-role-to-group edit dev-group
clusterrole.rbac.authorization.k8s.io/edit added: "dev-group"
[mahsan@r9 tmp]$ oc project
Using project "auth-rbac" on server "https://api.crc.testing:6443".
[mahsan@r9 tmp]$ oc policy add-role-to-group view da-group
clusterrole.rbac.authorization.k8s.io/view added: "da-group"
[mahsan@r9 tmp]$


[mahsan@r9 tmp]$ oc get rolebinding -o wide
NAME                    ROLE                               AGE    USERS   GROUPS                             SERVICEACCOUNTS
admin                   ClusterRole/admin                  2m2s   admin
<b>edit                    ClusterRole/edit                   76s            dev-group </b>
system:deployers        ClusterRole/system:deployer        2m2s                                              auth-rbac/deployer
system:image-builders   ClusterRole/system:image-builder   2m2s                                              auth-rbac/builder
system:image-pullers    ClusterRole/system:image-puller    2m2s           system:serviceaccounts:auth-rbac
<b>view                    ClusterRole/view                   42s            da-group</b>
[mahsan@r9 tmp]$


[mahsan@r9 tmp]$ oc adm groups add-users dev-group john
group.user.openshift.io/dev-group added: "john"
[mahsan@r9 tmp]$ oc get goups
error: the server doesn't have a resource type "goups"
[mahsan@r9 tmp]$ oc get groups.user.openshift.io
NAME        USERS
da-group    natasha
dev-group   developer, john
[mahsan@r9 tmp]$ oc login -u john -p redhat
Login successful.

You have one project on this server: "auth-rbac"

Using project "auth-rbac".

[mahsan@r9 tmp]$ oc project
Using project "auth-rbac" on server "https://api.crc.testing:6443".
[mahsan@r9 tmp]$ oc new-app --name mysql2 --image registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47
warning: Cannot find git. Ensure that it is installed and in your path. Git is required to work with git repositories.
--> Found container image 77d20f2 (5 years old) from registry.access.redhat.com for "registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47"

    MySQL 5.7
    ---------
    MySQL is a multi-user, multi-threaded SQL database server. The container image provides a containerized packaging of the MySQL mysqld daemon and client application. The mysqld server daemon accepts connections from clients and provides access to content from MySQL databases on behalf of the clients.

    Tags: database, mysql, mysql57, rh-mysql57

    * An image stream tag will be created as "mysql2:5.7-47" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "mysql2" created
    deployment.apps "mysql2" created
    service "mysql2" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/mysql2'
    Run 'oc status' to view your app.
[mahsan@r9 tmp]$



</pre>

