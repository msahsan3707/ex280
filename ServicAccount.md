<pre>
<h1> Service Account </h1>
[mahsan@r9 ~]$ oc new-project quad
Now using project "quad" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

<b>[mahsan@r9 ~]$ oc new-app --name wordpress-sa --docker-image wordpress </b>
Flag --docker-image has been deprecated, Deprecated flag use --image
warning: Cannot find git. Ensure that it is installed and in your path. Git is required to work with git repositories.
--> Found container image 29e1d31 (8 weeks old) from Docker Hub for "wordpress"

    * An image stream tag will be created as "wordpress-sa:latest" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "wordpress-sa" created
    deployment.apps "wordpress-sa" created
    service "wordpress-sa" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/wordpress-sa'
    Run 'oc status' to view your app.
[mahsan@r9 ~]$


[mahsan@r9 ~]$ oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                                READY   STATUS   RESTARTS      AGE
pod/wordpress-sa-689f7745b9-jpdnd   0/1     Error    2 (23s ago)   89s

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/wordpress-sa   ClusterIP   10.217.5.244   <none>        80/TCP    91s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wordpress-sa   0/1     1            0           91s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/wordpress-sa-64f76fccff   1         0         0       91s
replicaset.apps/wordpress-sa-689f7745b9   1         1         0       89s

NAME                                          IMAGE REPOSITORY                                                            TAGS     UPDATED
imagestream.image.openshift.io/wordpress-sa   default-route-openshift-image-registry.apps-crc.testing/quad/wordpress-sa   latest   About a minute ago
<b>[mahsan@r9 ~]$ oc logs pod/wordpress-sa-689f7745b9-jpdnd </b>
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 10.217.0.62. Set the 'ServerName' directive globally to suppress this message
(13)Permission denied: AH00072: make_sock: could not bind to address [::]:80
(13)Permission denied: AH00072: make_sock: could not bind to address 0.0.0.0:80
no listening sockets available, shutting down
AH00015: Unable to open logs
[mahsan@r9 ~]$


<b>[mahsan@r9 ~]$ oc create sa wordpress-sa </b>
serviceaccount/wordpress-sa created
[mahsan@r9 ~]$ oc get sa
NAME           SECRETS   AGE
builder        1         8m22s
default        1         8m22s
deployer       1         8m22s
wordpress-sa   1         3s
<b>[mahsan@r9 ~]$ oc adm policy add-scc-to-user anyuid -z wordpress-sa</b>
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "wordpress-sa"
[mahsan@r9 ~]$ oc get pods
NAME                            READY   STATUS             RESTARTS      AGE
wordpress-sa-689f7745b9-jpdnd   0/1     CrashLoopBackOff   5 (55s ago)   5m17s
[mahsan@r9 ~]$

<b>[mahsan@r9 ~]$ oc set serviceaccount deployment wordpress-sa wordpress-sa </b>
deployment.apps/wordpress-sa serviceaccount updated

[mahsan@r9 ~]$ oc get pods
NAME                            READY   STATUS    RESTARTS   AGE
wordpress-sa-54b9748789-5qp4t   1/1     Running   0          11s


<h1> Example #2 </h1>

[mahsan@r9 ~]$ oc new-project appsec-scc
Now using project "appsec-scc" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

<b> mahsan@r9 ~]$ oc new-app --name nginx --image=nginx </b>
warning: Cannot find git. Ensure that it is installed and in your path. Git is required to work with git repositories.
--> Found container image 4cad75a (2 months old) from Docker Hub for "nginx"

    * An image stream tag will be created as "nginx:latest" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "nginx" created
    deployment.apps "nginx" created
    service "nginx" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/nginx'
    Run 'oc status' to view your app.
[mahsan@r9 ~]$ oc get pods
NAME                    READY   STATUS              RESTARTS   AGE
nginx-5474fbd7f-2dk4n   0/1     ContainerCreating   0          2s
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc get pods
NAME                    READY   STATUS             RESTARTS      AGE
nginx-5474fbd7f-2dk4n   0/1     CrashLoopBackOff   1 (15s ago)   26s
[mahsan@r9 ~]$ oc logs nginx-5474fbd7f-2dk4n
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: can not modify /etc/nginx/conf.d/default.conf (read-only file system?)
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/04/10 20:28:59 [warn] 1#1: the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
nginx: [warn] the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
2025/04/10 20:28:59 [emerg] 1#1: mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
[mahsan@r9 ~]$

<b>[mahsan@r9 ~]$ oc create sa nginx-sa </b>
serviceaccount/nginx-sa created
[mahsan@r9 ~]$ oc get sa
NAME       SECRETS   AGE
builder    1         3m10s
default    1         3m10s
deployer   1         3m10s
nginx-sa   1         4s

<b> [mahsan@r9 ~]$ oc adm policy add-scc-to-user anyuid -z nginx-sa </b>
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "nginx-sa"
[mahsan@r9 ~]$

deployment.apps/nginx serviceaccount updated
[mahsan@r9 ~]$ oc get pods
NAME                    READY   STATUS    RESTARTS   AGE
nginx-f5c76b9c5-522tq   1/1     Running   0          6s
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc expose svc nginx
route.route.openshift.io/nginx exposed
[mahsan@r9 ~]$ oc get svc
NAME    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   10.217.5.89   <none>        80/TCP    4m30s
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc get routes.route.openshift.io
NAME    HOST/PORT                           PATH   SERVICES   PORT     TERMINATION   WILDCARD
nginx   nginx-appsec-scc.apps-crc.testing          nginx      80-tcp                 None
[mahsan@r9 ~]$ oc delete routes.route.openshift.io nginx
route.route.openshift.io "nginx" deleted
[mahsan@r9 ~]$


</pre>

