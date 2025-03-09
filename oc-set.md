<pre>
<h1> oc set storage / volume </h1>

[mahsan@r9 ~]$ oc new-project authorization-secrets
Now using project "authorization-secrets" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname



[mahsan@r9 ~]$ oc create secret generic mysql-p4 --from-literal user=myuser --from-literal password=redhat123 --from-literal database=test_secrets --from-literal hostname=mysql
secret/mysql-p4 created
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc new-app --name mysql-p4 --docker-image registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47
Flag --docker-image has been deprecated, Deprecated flag use --image
warning: Cannot find git. Ensure that it is installed and in your path. Git is required to work with git repositories.
--> Found container image 77d20f2 (5 years old) from registry.access.redhat.com for "registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47"

    MySQL 5.7
    ---------
    MySQL is a multi-user, multi-threaded SQL database server. The container image provides a containerized packaging of the MySQL mysqld daemon and client application. The mysqld server daemon accepts connections from clients and provides access to content from MySQL databases on behalf of the clients.

    Tags: database, mysql, mysql57, rh-mysql57

    * An image stream tag will be created as "mysql-p4:5.7-47" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "mysql-p4" created
    deployment.apps "mysql-p4" created
    service "mysql-p4" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/mysql-p4'
    Run 'oc status' to view your app.
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc get pods -w
NAME                       READY   STATUS   RESTARTS   AGE
mysql-p4-7475bb9c6-8kqzm   0/1     Error    0          29s
mysql-p4-7475bb9c6-8kqzm   0/1     Error    1 (1s ago)   29s
mysql-p4-7475bb9c6-8kqzm   0/1     CrashLoopBackOff   1 (2s ago)   30s
[mahsan@r9 ~]$ oc logs mysql-p4-7475bb9c6-8kqzm
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

[mahsan@r9 ~]$ oc set env deployment mysql-p4 --from secret/mysql-p4 --prefix MYSQL_
deployment.apps/mysql-p4 updated
[mahsan@r9 ~]$ oc get pods -w
NAME                       READY   STATUS    RESTARTS   AGE
mysql-p4-5788bdb5c-gkhsg   1/1     Running   0          3s
[mahsan@r9 ~]$ oc get pods
NAME                       READY   STATUS    RESTARTS   AGE
mysql-p4-5788bdb5c-gkhsg   1/1     Running   0          8s
[mahsan@r9 ~]$


[mahsan@r9 ~]$ oc set volume deployment/mysql-p4 --add --type secret --mount-path /run/secret/mysql-p4 --secret-name mysql-p4
info: Generated volume name: volume-57s4t
deployment.apps/mysql-p4 volume updated
[mahsan@r9 ~]$ oc get pods
NAME                        READY   STATUS    RESTARTS   AGE
mysql-p4-6c74f6bc87-j7cz5   1/1     Running   0          7s
[mahsan@r9 ~]$ oc get pods
NAME                        READY   STATUS    RESTARTS   AGE
mysql-p4-6c74f6bc87-j7cz5   1/1     Running   0          13s
[mahsan@r9 ~]$ oc get pods
NAME                        READY   STATUS    RESTARTS   AGE
mysql-p4-6c74f6bc87-j7cz5   1/1     Running   0          113s
[mahsan@r9 ~]$


spec:
      containers:
      - env:
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              key: database
              name: mysql-p4
        - name: MYSQL_HOSTNAME
          valueFrom:
            secretKeyRef:
              key: hostname
              name: mysql-p4
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: mysql-p4
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              key: user
              name: mysql-p4
        image: registry.access.redhat.com/rhscl/mysql-57-rhel7@sha256:225ccc7a059a8e615ce3e5f4aa6fae507a9e03b4562dccf3a98b09ff69bcdeb7
        imagePullPolicy: IfNotPresent
        name: mysql-p4
        ports:
        - containerPort: 3306
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /run/secret/mysql-p4
          name: volume-57s4t
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - name: volume-57s4t
        secret:
          defaultMode: 420
          secretName: mysql-p4




</pre>


