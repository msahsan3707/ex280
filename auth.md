<pre>
<h1> Auth </h1>

[mahsan@r9 ~]$ htpasswd -c -b -B /tmp/htpasswd admin redhat
Adding password for user admin
[mahsan@r9 ~]$ htpasswd -c -b -B /tmp/htpasswd indionce redhat
Adding password for user indionce
[mahsan@r9 ~]$ htpasswd -c -b -B /tmp/htpasswd qwerty redhat
Adding password for user qwerty
[mahsan@r9 ~]$ htpasswd -c -b -B /tmp/htpasswd john  redhat
Adding password for user john
[mahsan@r9 ~]$ htpasswd -c -b -B /tmp/htpasswd admin redhat
Adding password for user admin
[mahsan@r9 ~]$ htpasswd  -b -B /tmp/htpasswd indionce redhat
Adding password for user indionce
[mahsan@r9 ~]$ htpasswd  -b -B /tmp/htpasswd qwerty redhat
Adding password for user qwerty
[mahsan@r9 ~]$ htpasswd  -b -B /tmp/htpasswd john redhat
Adding password for user john
[mahsan@r9 ~]$ htpasswd  -b -B /tmp/htpasswd natasha redhat
Adding password for user natasha
[mahsan@r9 ~]$ htpasswd  -b -B /tmp/htpasswd harry redhat
Adding password for user harry
[mahsan@r9 ~]$ htpasswd  -b -B /tmp/htpasswd sarah redhat
Adding password for user sarah
[mahsan@r9 ~]$ htpasswd  -b -B /tmp/htpasswd mahsan redhat
Adding password for user mahsan
[mahsan@r9 ~]$ htpasswd  -b -B /tmp/htpasswd new_admin redhat
Adding password for user new_admin
[mahsan@r9 ~]$ htpasswd  -b -B /tmp/htpasswd new_developer redhat
Adding password for user new_developer

[mahsan@r9 ~]$ cat /tmp/htpasswd
admin:$2y$05$6uPxTlc.PeG.WFAM0layWObsxN1fI8BnDAlE2bOswNCr9U.ZPUkC.
indionce:$2y$05$H2o9X1pzkAQDMUy7VHQ1huJmBJ3lh58OTkeUH69blmw9UP9Hb7mGi
qwerty:$2y$05$mZyHbzvs0ZMrGfL6EqEcJOu9HBF42aFGrC6vf6TKuYDCUsrh/X7Bm
john:$2y$05$dUnKT4BAn3PVmk.5cYReXegmLOVolzQySVUdbEpKOQfotPSs.wwM6
natasha:$2y$05$9KWP4skcCHcoMZM6PtvVduvAq9qcuYRAkkHZ/tfCpSnL1dG8uLKUC
harry:$2y$05$gt/8QZZcNHstYPggde6NH.TfevJGrNWJ5DCl1LOZMSGJDXKvnIe/6
sarah:$2y$05$FEPvAINsqqf6J5qAalc83OOn.C1BhXn59nWyCnfFHdzFPC7cnphKK
mahsan:$2y$05$KzVD6MV.Rfk9JlhBENRQMOXF7dTwf9snYqsbIO8/X.8o/FzoHjCIS
new_admin:$2y$05$vozLQXPF2xi5cHlq5DAd/ef6YNr2zyxrt6J/4BCU22U4EBZEczq/m
new_developer:$2y$05$dKwhTBlZvRH18s5bjw8TGesFOaDPWEhYmGGMv/zHnG.eJkU7Xf7YW
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc create secret generic localusers --from-file htpasswd=/tmp/htpasswd -n openshift-config
secret/localusers created
[mahsan@r9 ~]$ oc get secrets -n openshift-config
NAME                                      TYPE                             DATA   AGE
builder-dockercfg-7r7fz                   kubernetes.io/dockercfg          1      92d
default-dockercfg-676lc                   kubernetes.io/dockercfg          1      92d
deployer-dockercfg-566hs                  kubernetes.io/dockercfg          1      92d
etcd-client                               kubernetes.io/tls                2      93d
htpass-secret                             Opaque                           1      92d
initial-service-account-private-key       Opaque                           1      93d
localusers                                Opaque                           1      14s
login-template                            Opaque                           1      92d
pull-secret                               kubernetes.io/dockerconfigjson   1      93d
webhook-authentication-integrated-oauth   Opaque                           1      93d
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc create secret generic localusers --from-file htpasswd=/tmp/htpasswd -n openshift-config
secret/localusers created
[mahsan@r9 ~]$ oc get secrets -n openshift-config
NAME                                      TYPE                             DATA   AGE
builder-dockercfg-7r7fz                   kubernetes.io/dockercfg          1      92d
default-dockercfg-676lc                   kubernetes.io/dockercfg          1      92d
deployer-dockercfg-566hs                  kubernetes.io/dockercfg          1      92d
etcd-client                               kubernetes.io/tls                2      93d
htpass-secret                             Opaque                           1      92d
initial-service-account-private-key       Opaque                           1      93d
localusers                                Opaque                           1      14s
login-template                            Opaque                           1      92d
pull-secret                               kubernetes.io/dockerconfigjson   1      93d
webhook-authentication-integrated-oauth   Opaque               

[mahsan@r9 ~]$ oc adm policy add-cluster-role-to-user cluster-admin admin
Warning: User 'admin' not found
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "admin"
[mahsan@r9 ~]$ oc adm policy add-cluster-role-to-user cluster-admin admin
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "admin"
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc get oauth cluster -o yaml > /tmp/oauth.yaml

[mahsan@r9 ~]$ oc get oauth cluster -o yaml > /tmp/oauth.yaml
[mahsan@r9 ~]$ vim /tmp/oauth.yaml
[mahsan@r9 ~]$ tail -20 /tmp/oauth.yaml
  ownerReferences:
  - apiVersion: config.openshift.io/v1
    kind: ClusterVersion
    name: version
    uid: 1de19896-228b-4a9d-913f-0d775bc4bebc
  resourceVersion: "20146"
  uid: 34972387-914d-47a6-8643-c440983c06f2
spec:
  identityProviders:
  - htpasswd:
      fileData:
        name: localusers
    mappingMethod: claim
    name: myusers
    type: HTPasswd
  templates:
    login:
      name: login-template
  tokenConfig:
    accessTokenMaxAgeSeconds: 0
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc replace -f /tmp/oauth.yaml -n openshift-config
oauth.config.openshift.io/cluster replaced
[mahsan@r9 ~]$ oc get pod -n openshift-authentication -w
NAME                              READY   STATUS    RESTARTS   AGE
oauth-openshift-b468b8d84-nwchh   1/1     Running   2          71d
oauth-openshift-b468b8d84-nwchh   1/1     Terminating   2          71d
oauth-openshift-7cb76bf4c6-42gs5   0/1     Pending       0          0s
oauth-openshift-7cb76bf4c6-42gs5   0/1     Pending       0          0s
oauth-openshift-b468b8d84-nwchh    0/1     Terminating   2          71d
oauth-openshift-7cb76bf4c6-42gs5   0/1     Pending       0          26s
oauth-openshift-7cb76bf4c6-42gs5   0/1     Pending       0          26s
oauth-openshift-7cb76bf4c6-42gs5   0/1     ContainerCreating   0          26s
oauth-openshift-7cb76bf4c6-42gs5   0/1     ContainerCreating   0          26s
oauth-openshift-7cb76bf4c6-42gs5   0/1     Running             0          27s
oauth-openshift-b468b8d84-nwchh    0/1     Terminating         2          71d
oauth-openshift-b468b8d84-nwchh    0/1     Terminating         2          71d
oauth-openshift-7cb76bf4c6-42gs5   1/1     Running             0          28s
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc login -u admin -p redhat
Login successful.

You have access to 68 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "volume2".
[mahsan@r9 ~]$ oc get node
NAME   STATUS   ROLES                         AGE   VERSION
crc    Ready    control-plane,master,worker   93d   v1.30.6
[mahsan@r9 ~]$ oc login -u mahsan -p redhat
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>

[mahsan@r9 ~]$ oc get node
Error from server (Forbidden): nodes is forbidden: User "mahsan" cannot list resource "nodes" in API group "" at the cluster scope
[mahsan@r9 ~]$

<h2><b> add and remove users </b></h2>

[mahsan@r9 ~]$ oc extract secret/localusers -n openshift-config --to /tmp/ --confirm
/tmp/htpasswd
[mahsan@r9 ~]$ ls -ltr /tmp/
total 8
drwx------ 2 root   root      6 Mar  5 14:00 vmware-root_1034-2990678769
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-dbus-broker.service-6hPyTd
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-bluetooth.service-u0gMJB
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-chronyd.service-NEKiOs
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-irqbalance.service-6sz14B
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-power-profiles-daemon.service-z7vDCL
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-rtkit-daemon.service-PuKzuX
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-switcheroo-control.service-CHExGO
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-systemd-logind.service-XINl3o
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-upower.service-XRz4r7
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-ModemManager.service-I6lrvU
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-kdump.service-Qomu44
drwx------ 3 root   root     17 Mar  6 09:11 systemd-private-85764c3d0df847abac913e65df4a8933-colord.service-qMbCNM
drwx------ 3 root   root     17 Mar  6 10:29 systemd-private-85764c3d0df847abac913e65df4a8933-fwupd.service-5B4BQy
drwx------ 2 mahsan mahsan   24 Mar  6 10:29 ssh-XXXX5zXZOY
-rw-r--r-- 1 mahsan mahsan 1184 Mar  6 10:52 oauth.yaml
-rw-r--r-- 1 mahsan mahsan  688 Mar  6 11:09 htpasswd
[mahsan@r9 ~]$


<b> mahsan@r9 ~]$ htpasswd -b /tmp/htpasswd mahsan tripod001 </b>
Updating password for user mahsan
[mahsan@r9 ~]$ oc set data secret/localusers --from-file htpasswd=/tmp/htpasswd -n openshift-config
secret/localusers data updated
[mahsan@r9 ~]$ oc get pod -n openshift-authentication -w
NAME                               READY   STATUS    RESTARTS   AGE
oauth-openshift-7cb76bf4c6-42gs5   1/1     Running   0          12m

[mahsan@r9 ~]$ oc login -u mahsan -p redhat
Login failed (401 Unauthorized)
Verify you have provided the correct credentials.
[mahsan@r9 ~]$ oc login -u mahsan -p tripod001
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>

[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc extract secret/localusers -n openshift-config --to /tmp/ --confirm
/tmp/htpasswd
[mahsan@r9 ~]$ htpasswd -D /tmp/htpasswd mahsan
Deleting password for user mahsan
[mahsan@r9 ~]$ cat /tmp/htpasswd
admin:$2y$05$6uPxTlc.PeG.WFAM0layWObsxN1fI8BnDAlE2bOswNCr9U.ZPUkC.
indionce:$2y$05$H2o9X1pzkAQDMUy7VHQ1huJmBJ3lh58OTkeUH69blmw9UP9Hb7mGi
qwerty:$2y$05$mZyHbzvs0ZMrGfL6EqEcJOu9HBF42aFGrC6vf6TKuYDCUsrh/X7Bm
john:$2y$05$dUnKT4BAn3PVmk.5cYReXegmLOVolzQySVUdbEpKOQfotPSs.wwM6
natasha:$2y$05$9KWP4skcCHcoMZM6PtvVduvAq9qcuYRAkkHZ/tfCpSnL1dG8uLKUC
harry:$2y$05$gt/8QZZcNHstYPggde6NH.TfevJGrNWJ5DCl1LOZMSGJDXKvnIe/6
sarah:$2y$05$FEPvAINsqqf6J5qAalc83OOn.C1BhXn59nWyCnfFHdzFPC7cnphKK
new_admin:$2y$05$vozLQXPF2xi5cHlq5DAd/ef6YNr2zyxrt6J/4BCU22U4EBZEczq/m
new_developer:$2y$05$dKwhTBlZvRH18s5bjw8TGesFOaDPWEhYmGGMv/zHnG.eJkU7Xf7YW
[mahsan@r9 ~]$ oc set data secret/localusers --from-file htpasswd=/tmp/htpasswd -n openshift-config
secret/localusers data updated
[mahsan@r9 ~]$ oc get users
NAME        UID                                    FULL NAME   IDENTITIES
admin       6ca28dbe-b8fe-4010-820f-2159648d3de8               myusers:admin
developer   6e16cbbe-41aa-49cd-8158-5aab2967cadf               developer:developer
kubeadmin   42c8dcb9-09ea-4b49-898c-4d90aad72acd               developer:kubeadmin
mahsan      e1c68c21-840c-45c4-8668-ea0d1405368e               myusers:mahsan

[mahsan@r9 ~]$ oc login -u mahsan -p tripod001
Login failed (401 Unauthorized)
Verify you have provided the correct credentials.
[mahsan@r9 ~]$
<b> Delete User and Identity </b>
[mahsan@r9 ex280]$ oc get identities.user.openshift.io
NAME                  IDP NAME    IDP USER NAME   USER NAME   USER UID
developer:developer   developer   developer       developer   6e16cbbe-41aa-49cd-8158-5aab2967cadf
developer:kubeadmin   developer   kubeadmin       kubeadmin   42c8dcb9-09ea-4b49-898c-4d90aad72acd
myusers:admin         myusers     admin           admin       6ca28dbe-b8fe-4010-820f-2159648d3de8
myusers:mahsan        myusers     mahsan          mahsan      e1c68c21-840c-45c4-8668-ea0d1405368e
[mahsan@r9 ex280]$ oc get users
NAME        UID                                    FULL NAME   IDENTITIES
admin       6ca28dbe-b8fe-4010-820f-2159648d3de8               myusers:admin
developer   6e16cbbe-41aa-49cd-8158-5aab2967cadf               developer:developer
kubeadmin   42c8dcb9-09ea-4b49-898c-4d90aad72acd               developer:kubeadmin
mahsan      e1c68c21-840c-45c4-8668-ea0d1405368e               myusers:mahsan
[mahsan@r9 ex280]$ oc delete user mahsan
user.user.openshift.io "mahsan" deleted
[mahsan@r9 ex280]$ oc delete identities.user.openshift.io myusers:mahsan
identity.user.openshift.io "myusers:mahsan" deleted
[mahsan@r9 ex280]$ oc get users
NAME        UID                                    FULL NAME   IDENTITIES
admin       6ca28dbe-b8fe-4010-820f-2159648d3de8               myusers:admin
developer   6e16cbbe-41aa-49cd-8158-5aab2967cadf               developer:developer
kubeadmin   42c8dcb9-09ea-4b49-898c-4d90aad72acd               developer:kubeadmin
[mahsan@r9 ex280]$ oc get identities.user.openshift.io
NAME                  IDP NAME    IDP USER NAME   USER NAME   USER UID
developer:developer   developer   developer       developer   6e16cbbe-41aa-49cd-8158-5aab2967cadf
developer:kubeadmin   developer   kubeadmin       kubeadmin   42c8dcb9-09ea-4b49-898c-4d90aad72acd
myusers:admin         myusers     admin           admin       6ca28dbe-b8fe-4010-820f-2159648d3de8
[mahsan@r9 ex280]$ oc delete secrets localusers -n openshift-config
secret "localusers" deleted
[mahsan@r9 ex280]$
 

</pre>

