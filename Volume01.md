<pre>

<h1> Volume </h1> 

[mahsan@r9 ~]$ oc new-app --name postgresql-persistent --docker-image registry.redhat.io/rhel8/postgresql-12:1-43 \
> -e POSTGRESQL_USER=redhat  -e POSTGRESQL_PASSWORD=redhat123 -e POSTGRESQL_DATABASE=persistentdb
Flag --docker-image has been deprecated, Deprecated flag use --image
warning: Cannot find git. Ensure that it is installed and in your path. Git is required to work with git repositories.
--> Found container image 667898a (4 years old) from registry.redhat.io for "registry.redhat.io/rhel8/postgresql-12:1-43"

    PostgreSQL 12
    -------------
    PostgreSQL is an advanced Object-Relational database management system (DBMS). The image contains the client and server programs that you'll need to create, run, maintain and access a PostgreSQL DBMS server.

    Tags: database, postgresql, postgresql12, postgresql-12

    * An image stream tag will be created as "postgresql-persistent:1-43" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "postgresql-persistent" created
    deployment.apps "postgresql-persistent" created
    service "postgresql-persistent" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/postgresql-persistent'
    Run 'oc status' to view your app.
[mahsan@r9 ~]$ oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                                         READY   STATUS              RESTARTS   AGE
pod/postgresql-persistent-69b49d57d9-bf29l   0/1     ContainerCreating   0          9s

NAME                            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/postgresql-persistent   ClusterIP   10.217.5.61   <none>        5432/TCP   11s

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgresql-persistent   0/1     1            0           11s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/postgresql-persistent-69b49d57d9   1         1         0       9s
replicaset.apps/postgresql-persistent-69b5fd59b8   1         0         0       11s

NAME                                                   IMAGE REPOSITORY                                                                       TAGS   UPDATED
imagestream.image.openshift.io/postgresql-persistent   default-route-openshift-image-registry.apps-crc.testing/volume/postgresql-persistent   1-43   9 seconds ago
[mahsan@r9 ~]$ oc get pods -w
NAME                                     READY   STATUS              RESTARTS   AGE
postgresql-persistent-69b49d57d9-bf29l   0/1     ContainerCreating   0          17s
postgresql-persistent-69b49d57d9-bf29l   1/1     Running             0          34s
[mahsan@r9 ~]$

[mahsan@r9 ~]$  oc set volume deployment postgresql-persistent --add --name postgresql-storage -t pvc --claim-size 10G --claim-class crc-csi-hostpath-provisioner --claim-mode rwo \
> --claim-name postgresql-storage -m /var/lib/psql
deployment.apps/postgresql-persistent volume updated
[mahsan@r9 ~]$ oc get pods
NAME                                    READY   STATUS    RESTARTS   AGE
postgresql-persistent-9d6cb7789-r47w7   1/1     Running   0          3m27s
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc get pvc
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS                   VOLUMEATTRIBUTESCLASS   AGE
postgresql-storage   Bound    pvc-88e433fb-0035-430c-825e-0d563d461eb7   30Gi       RWO            crc-csi-hostpath-provisioner   <unset>                 4m54s
[mahsan@r9 ~]$ oc get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                 STORAGECLASS                   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-017a3de9-f3b0-41bb-a84e-64c62f4b3f9b   30Gi       RWX            Retain           Bound    openshift-image-registry/crc-image-registry-storage   crc-csi-hostpath-provisioner   <unset>                          91d
pvc-88e433fb-0035-430c-825e-0d563d461eb7   30Gi       RWO            Retain           Bound    volume/postgresql-storage                             crc-csi-hostpath-provisioner   <unset>                          4m44s
[mahsan@r9 ~]$

<h1> Volume2 </h1>

[mahsan@r9 ~]$ oc new-app --name postgresql-persistent2 \
> --docker-image registry.redhat.io/rhel8/postgresql-12:1-43 \
> -e POSTGRESQL_USER=redhat \
> -e POSTGRESQL_PASSWORD=redhat123 \
> -e POSTGRESQL_DATABASE=persistentdb
Flag --docker-image has been deprecated, Deprecated flag use --image
--> Found container image 667898a (4 years old) from registry.redhat.io for "registry.redhat.io/rhel8/postgresql-12:1-43"

    PostgreSQL 12
    -------------
    PostgreSQL is an advanced Object-Relational database management system (DBMS). The image contains the client and server programs that you'll need to create, run, maintain and access a PostgreSQL DBMS server.

    Tags: database, postgresql, postgresql12, postgresql-12

    * An image stream tag will be created as "postgresql-persistent2:1-43" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "postgresql-persistent2" created
    deployment.apps "postgresql-persistent2" created
    service "postgresql-persistent2" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/postgresql-persistent2'
    Run 'oc status' to view your app.
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc set volumes deployment/postgresql-persistent2 --add --name postgresql-storage -t pvc \
> --claim-name postgresql-storage -m /var/lib/pgsql
deployment.apps/postgresql-persistent2 volume updated
[mahsan@r9 ~]$ oc get pods
NAME                                      READY   STATUS    RESTARTS   AGE
postgresql-persistent2-5bb8cd656b-xkh5x   0/1     Pending   0          11s
postgresql-persistent2-654d4cf855-jfvqt   1/1     Running   0          14m
[mahsan@r9 ~]$

 spec:
      containers:
      - env:
        - name: POSTGRESQL_DATABASE
          value: persistentdb
        - name: POSTGRESQL_PASSWORD
          value: redhat123
        - name: POSTGRESQL_USER
          value: redhat
        image: registry.redhat.io/rhel8/postgresql-12@sha256:aba696824c21625735a9b0140116df087401705156bc0e7efbf35b4e76127e6a
        imagePullPolicy: IfNotPresent
        name: postgresql-persistent2
        ports:
        - containerPort: 5432
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/lib/pgsql
          name: postgresql-storage
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - name: postgresql-storage
        persistentVolumeClaim:
          claimName: postgresql-storage
		  
		  



</pre>
