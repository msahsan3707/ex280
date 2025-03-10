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

</pre>

