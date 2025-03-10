<pre>
<h2> Word press </h2>

[mahsan@r9 ~]$ [mahsan@r9 ~]$ oc new-app --name gitlab --docker-image bitnami/gitlab-runner
Flag --docker-image has been deprecated, Deprecated flag use --image
--> Found container image 1d9ac0e (2 days old) from Docker Hub for "bitnami/gitlab-runner"

    * An image stream tag will be created as "gitlab:latest" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "gitlab" created
    deployment.apps "gitlab" created
--> Success
    Run 'oc status' to view your app.
[mahsan@r9 ~]$ oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                          READY   STATUS    RESTARTS   AGE
pod/gitlab-77fd4974db-pmbj9   1/1     Running   0          3s

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gitlab   1/1     1            1           4s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/gitlab-6fdfc6d84    0         0         0       4s
replicaset.apps/gitlab-77fd4974db   1         1         1       3s

NAME                                    IMAGE REPOSITORY                                                                       TAGS     UPDATED
imagestream.image.openshift.io/gitlab   default-route-openshift-image-registry.apps-crc.testing/authorization-secrets/gitlab   latest   3 seconds ago
[mahsan@r9 ~]$ oc get pod gitlab-77fd4974db-pmbj9 -o yaml | oc adm policy scc-subject-review -f -
RESOURCE                      ALLOWED BY
Pod/gitlab-77fd4974db-pmbj9   restricted-v2
[mahsan@r9 ~]$ oc create sa gitlab-sa
serviceaccount/gitlab-sa created
[mahsan@r9 ~]$ oc adm policy add-scc-to-user anyuid -z gitlab-sa
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "gitlab-sa"
[mahsan@r9 ~]$ oc get pod gitlab-77fd4974db-pmbj9 -o yaml | oc adm policy scc-subject-review -f -
RESOURCE                      ALLOWED BY
Pod/gitlab-77fd4974db-pmbj9   restricted-v2
[mahsan@r9 ~]$ oc set serviceaccount deployment/gitlab gitlab-sa
deployment.apps/gitlab serviceaccount updated
[mahsan@r9 ~]$ oc get deployment
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
gitlab   1/1     1            1           9m21s
[mahsan@r9 ~]$ oc get deployment gitlab -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "3"
    image.openshift.io/triggers: '[{"from":{"kind":"ImageStreamTag","name":"gitlab:latest"},"fieldPath":"spec.template.spec.containers[?(@.name==\"gitlab\")].image"}]'
    openshift.io/generated-by: OpenShiftNewApp
  creationTimestamp: "2025-03-10T21:26:16Z"
  generation: 3
  labels:
    app: gitlab
    app.kubernetes.io/component: gitlab
    app.kubernetes.io/instance: gitlab
  name: gitlab
  namespace: authorization-secrets
  resourceVersion: "88604"
  uid: 3505b185-646a-4b8a-8dd0-e460cdcf41f3
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      deployment: gitlab
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      annotations:
        openshift.io/generated-by: OpenShiftNewApp
      creationTimestamp: null
      labels:
        deployment: gitlab
    spec:
      containers:
      - image: bitnami/gitlab-runner@sha256:14c441341ffaf7641aeeb2cf858256a2d291ef39321b62bf54be2b9c4a96a416
        imagePullPolicy: IfNotPresent
        name: gitlab
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: gitlab-sa
      serviceAccountName: gitlab-sa
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2025-03-10T21:26:18Z"
    lastUpdateTime: "2025-03-10T21:26:18Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2025-03-10T21:26:16Z"
    lastUpdateTime: "2025-03-10T21:29:12Z"
    message: ReplicaSet "gitlab-55c67fc448" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 3
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
[mahsan@r9 ~]$ oc get svc
No resources found in authorization-secrets namespace.
[mahsan@r9 ~]$ oc expose deployment gitlab --port 8080
service/gitlab exposed
[mahsan@r9 ~]$ oc get svc
NAME     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
gitlab   ClusterIP   10.217.4.129   <none>        8080/TCP   9s
[mahsan@r9 ~]$ oc expose  svc gitlab
route.route.openshift.io/gitlab exposed
[mahsan@r9 ~]$ oc get routes.route.openshift.io
NAME     HOST/PORT                                       PATH   SERVICES   PORT   TERMINATION   WILDCARD
gitlab   gitlab-authorization-secrets.apps-crc.testing          gitlab     8080                 None
[mahsan@r9 ~]$ oc delete routes.route.openshift.io gitlab
route.route.openshift.io "gitlab" deleted
[mahsan@r9 ~]$ oc expose svc gitlab --port 8080 --hostname gitlab.apps-crc.testing
route.route.openshift.io/gitlab exposed
[mahsan@r9 ~]$ oc get routes.route.openshift.io
NAME     HOST/PORT                 PATH   SERVICES   PORT   TERMINATION   WILDCARD
gitlab   gitlab.apps-crc.testing          gitlab     8080                 None
[mahsan@r9 ~]$ curl -s http://gitlab.apps-crc.testing/users/sign_in | grep '<title>'
[mahsan@r9 ~]$ curl -s http://gitlab.apps-crc.testing/users/sign_in

 <b>MYSQL 2 </b>


[mahsan@r9 ~]$ oc new-project auth-review
Now using project "auth-review" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

[mahsan@r9 ex280]$ oc create secret generic review-secret --from-literal=user=wpuser --from-literal=password=redha123 --from-literal=database=wordpress
secret/review-secret created
[mahsan@r9 ex280]$ oc get secrets
NAME                       TYPE                      DATA   AGE
builder-dockercfg-sc657    kubernetes.io/dockercfg   1      7m54s
default-dockercfg-jkvtm    kubernetes.io/dockercfg   1      7m54s
deployer-dockercfg-lrzbc   kubernetes.io/dockercfg   1      7m54s
review-secret              Opaque                    3      5s
[mahsan@r9 ex280]$

[mahsan@r9 ex280]$ oc new-app mysql --docker-image registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47
Flag --docker-image has been deprecated, Deprecated flag use --image
--> Found container image 77d20f2 (5 years old) from registry.access.redhat.com for "registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47"

    MySQL 5.7
    ---------
    MySQL is a multi-user, multi-threaded SQL database server. The container image provides a containerized packaging of the MySQL mysqld daemon and client application. The mysqld server daemon accepts connections from clients and provides access to content from MySQL databases on behalf of the clients.

    Tags: database, mysql, mysql57, rh-mysql57

    * An image stream tag will be created as "mysql-57-rhel7:5.7-47" that will track this image

--> Found image 9314411 (3 months old) in image stream "openshift/mysql" under tag "8.0-el8" for "mysql"

    MySQL 8.0
    ---------
    MySQL is a multi-user, multi-threaded SQL database server. The container image provides a containerized packaging of the MySQL mysqld daemon and client application. The mysqld server daemon accepts connections from clients and provides access to content from MySQL databases on behalf of the clients.

    Tags: database, mysql, mysql80, mysql-80


--> Creating resources ...
    imagestream.image.openshift.io "mysql-57-rhel7" created
    deployment.apps "mysql-57-rhel7" created
    deployment.apps "mysql" created
    service "mysql-57-rhel7" created
    service "mysql" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/mysql-57-rhel7'
     'oc expose service/mysql'
    Run 'oc status' to view your app.
[mahsan@r9 ex280]$ oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                                 READY   STATUS              RESTARTS   AGE
pod/mysql-57-rhel7-f488c5f8d-dlt9f   0/1     ContainerCreating   0          6s
pod/mysql-74c9cbfc5c-w98nh           0/1     ContainerCreating   0          6s

NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/mysql            ClusterIP   10.217.4.91    <none>        3306/TCP   7s
service/mysql-57-rhel7   ClusterIP   10.217.5.193   <none>        3306/TCP   7s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql            0/1     1            0           7s
deployment.apps/mysql-57-rhel7   0/1     1            0           7s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-57-rhel7-7cd7f6d785   1         0         0       7s
replicaset.apps/mysql-57-rhel7-f488c5f8d    1         1         0       6s
replicaset.apps/mysql-698ffc9d7             1         0         0       7s
replicaset.apps/mysql-74c9cbfc5c            1         1         0       6s

NAME                                            IMAGE REPOSITORY                                                                     TAGS     UPDATED
imagestream.image.openshift.io/mysql-57-rhel7   default-route-openshift-image-registry.apps-crc.testing/auth-review/mysql-57-rhel7   5.7-47   6 seconds ago
[mahsan@r9 ex280]$

[mahsan@r9 ex280]$ oc logs pod/mysql-57-rhel7-f488c5f8d-dlt9fpod/mysql-57-rhel7-f488c5f8d-dlt9f
error: arguments in resource/name form may not have more than one slash
[mahsan@r9 ex280]$ oc logs pod/mysql-57-rhel7-f488c5f8d-dlt9f
Warning: Can't detect memory limit from cgroups
Warning: Can't detect number of CPU cores from cgroups
Warning: Can't detect memory limit from cgroups
Warning: Can't detect number of CPU cores from cgroups
=> sourcing 20-validate-variables.sh ...
You must either specify the following environment variables:
  MYSQL_USER (regex: '^[a-zA-Z0-9_]+$')
  MYSQL_PASSWORD (regex: '^[a-zA-Z0-9_~!@#$%^&*()-=<>,.?;:|]+$')
  MYSQL_DATABASE (regex: '^[a-zA-Z0-9_]+$')
Or the following environment variable:
  MYSQL_ROOT_PASSWORD (regex: '^[a-zA-Z0-9_~!@#$%^&*()-=<>,.?;:|]+$')
Or both.
Optional Settings:
  MYSQL_LOWER_CASE_TABLE_NAMES (default: 0)
  MYSQL_LOG_QUERIES_ENABLED (default: 0)
  MYSQL_MAX_CONNECTIONS (default: 151)
  MYSQL_FT_MIN_WORD_LEN (default: 4)
  MYSQL_FT_MAX_WORD_LEN (default: 20)
  MYSQL_AIO (default: 1)
  MYSQL_KEY_BUFFER_SIZE (default: 32M or 10% of available memory)
  MYSQL_MAX_ALLOWED_PACKET (default: 200M)
  MYSQL_TABLE_OPEN_CACHE (default: 400)
  MYSQL_SORT_BUFFER_SIZE (default: 256K)
  MYSQL_READ_BUFFER_SIZE (default: 8M or 5% of available memory)
  MYSQL_INNODB_BUFFER_POOL_SIZE (default: 32M or 50% of available memory)
  MYSQL_INNODB_LOG_FILE_SIZE (default: 8M or 15% of available memory)
  MYSQL_INNODB_LOG_BUFFER_SIZE (default: 8M or 15% of available memory)

For more information, see https://github.com/sclorg/mysql-container
[mahsan@r9 ex280]$ oc set env deployment mysql --from=secret/review-secret --prefix=MYSQL_
deployment.apps/mysql updated
[mahsan@r9 ex280]$ oc get pods -w
NAME                             READY   STATUS             RESTARTS        AGE
mysql-57-rhel7-f488c5f8d-dlt9f   0/1     Error              6 (2m59s ago)   7m11s
mysql-74c9cbfc5c-w98nh           0/1     CrashLoopBackOff   6 (28s ago)     7m11s
mysql-856fbc96bd-j4cgn           0/1     Pending            0               8s
mysql-57-rhel7-f488c5f8d-dlt9f   0/1     CrashLoopBackOff   6 (14s ago)     7m14s
mysql-856fbc96bd-j4cgn           0/1     Pending            0               22s
mysql-856fbc96bd-j4cgn           0/1     Pending            0               22s
mysql-856fbc96bd-j4cgn           0/1     ContainerCreating   0               22s
mysql-856fbc96bd-j4cgn           0/1     ContainerCreating   0               23s
mysql-856fbc96bd-j4cgn           1/1     Running             0               24s
mysql-74c9cbfc5c-w98nh           0/1     Terminating         6 (44s ago)     7m27s
mysql-74c9cbfc5c-w98nh           0/1     Terminating         6               7m27s
mysql-74c9cbfc5c-w98nh           0/1     Terminating         6               7m27s
mysql-74c9cbfc5c-w98nh           0/1     Terminating         6               7m28s
mysql-74c9cbfc5c-w98nh           0/1     Terminating         6               7m28s
[mahsan@r9 ex280]$ oc get pods
NAME                             READY   STATUS             RESTARTS      AGE
mysql-57-rhel7-f488c5f8d-dlt9f   0/1     CrashLoopBackOff   6 (76s ago)   8m16s
mysql-856fbc96bd-j4cgn           1/1     Running            0             73s
[mahsan@r9 ex280]$




</pre>

