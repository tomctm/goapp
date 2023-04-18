# FACEIT DevOps Challenge


Delivering application in GCP using terraform, I have chosen to set up a kubernetes cluster,  the folder gke-cluster have the content about the deploy.

```
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

kubernetes_cluster_host = "xxxxx"
kubernetes_cluster_name = "xxxxxx-cacao-384010-gke"
project_id = "xxxxxx-cacao-384010"
region = "us-xxxxx"


NAME                                                  STATUS   ROLES    AGE     VERSION
gke-effective-cacao--effective-cacao--dbb7f447-7fnv   Ready    <none>   5m40s   v1.24.10-gke.2300
gke-effective-cacao--effective-cacao--dbb7f447-tnzx   Ready    <none>   5m41s   v1.24.10-gke.2300

``````
Installing the PostgreSQL deployment with helm using the default namespace:

`````
NAME               READY   STATUS    RESTARTS   AGE
pod/postgresql-0   0/1     Running   0          27s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes      ClusterIP   10.147.240.1    <none>        443/TCP    18m
service/postgresql      ClusterIP   10.147.241.69   <none>        5432/TCP   28s
service/postgresql-hl   ClusterIP   None            <none>        5432/TCP   28s

NAME                          READY   AGE
statefulset.apps/postgresql   0/1     29s


- logs about postgress pod:

2023-04-18 06:00:27.472 GMT [1] LOG:  starting PostgreSQL 15.2 on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
2023-04-18 06:00:27.473 GMT [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2023-04-18 06:00:27.474 GMT [1] LOG:  listening on IPv6 address "::", port 5432
2023-04-18 06:00:27.479 GMT [1] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2023-04-18 06:00:27.486 GMT [130] LOG:  database system was shut down at 2023-04-18 06:00:27 GMT
2023-04-18 06:00:27.495 GMT [1] LOG:  database system is ready to accept connections
`````


- Test if we can connet to postgree database:

```` 

export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)



kubectl run postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:15.2.0-debian-11-r21 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host postgresql -U postgres -d postgres -p 5432
If you don't see a command prompt, try pressing enter.

postgres=#

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "postgresql" (address "10.147.241.69") at port "5432".

`````

Regarding about golang app, i created a dockerfile to build and push de app to dockerhub, you can see the Dockerfile into the test-app folder:

![Captura de Pantalla 2023-04-18 a las 8 06 31](https://user-images.githubusercontent.com/8587416/232686079-ce48f45b-a555-4cd4-8d06-0e82631fdb77.png)



i have been created the deployment in kubernetes in the deploy-app folder:

`````
NAME                         READY   STATUS             RESTARTS     AGE
go-web-app-9dcb6c569-bs9s8   0/1     CrashLoopBackOff   1 (5s ago)   8s
go-web-app-9dcb6c569-zzs24   0/1     CrashLoopBackOff   1 (5s ago)   8s

`````

**I have encountered an application crash that I have not been able to solve, since the connection port variable against postgress is not able to convert it to int. Honestly I don't have a debugging experience in GO that allows me to find the specific error. I have tested that the database allows connections and it is up.

```
time="2023-04-18T06:09:55Z" level=fatal msg="error running application" error="failed to process environment variables: envconfig.Process: assigning POSTGRESQL_PORT to PostgresPort: converting 'tcp://10.147.241.69:5432' to type int. details: strconv.ParseInt: parsing \"tcp://10.147.241.69:5432\": invalid syntax"
`````

Regarding to the HA availability, i think we use a architecture like this diagram:

![postgresql_architecture](https://user-images.githubusercontent.com/8587416/232689477-304ed716-b02d-41c2-aac0-702a39127d63.svg)


Regarding to the HA in golang app, apart from controlling the replicas through the deployment that we have in progress, it could be established in this deployment a mechanism to discover the values consumed by the pod, if they are exceeded, it could raise another pod at the moment.


On the integration a flow of CI/CD in this deployment we could use the following:

- CI/CD to the terraform delivery with this steps:
  - terraform plan stage
  - stage that you see the cost or the or the increased cost in case you add new infrastructure (tools like infracost for ex.)
  - manually stage to apply the changes
- CI/CD to the goland application
  - stage to build the app
  - stage to create a new tag
  - stage to push the artifact to dockerhub or application that you use like repository of artifacts
In this scenario when we apply changes with a PullRequest or Merge Request in main branch, trigger the workflow acording to the folder that you applied the changes.


  - 
