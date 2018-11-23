# Deployment scripts for sqlserver and dotnet core app in kubernetes

## Create a Persistent Volume
Persistent volume claim is needed to store SQL Server data and yaml snippet to create a 5 GB storage is displayed below. The deployment file is going to mount files to this storage claim. You can read more about Persistent Volumes.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mssql-sample-data-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
   requests:
    storage: 5Gi
```
## Create a Kubernetes Secret
The password for sa user will be created as a Kubernetes Secret. Please replace 'UEBzc3dvcmQxJA==' password with actual base64 password you intend to specify for your SQL Server instance. Your can read more about Secrets. 

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: mssql-sample-secret
  namespace: default
data:
  # Password is P@ssword1$ so update it with password of your choice  
  SA_PASSWORD: UEBzc3dvcmQxJA==
type: Opaque
```

## Create a Kubernetes Service for sql server
The next step is to create a Kubernetes Service for SQL Server. As you can see in yaml snippet below, port 1433 is used and type is ClusterIP i.e. this service doesn't has external endpoints. Kubernetes will use to selector 'app: mssql-sample' to map to the deployment as you are going to see next. You can read more about Services. 

`sqlserver.svc.yaml`

## Create a Deployment for sql server
The next step is to create a SQL Server deployment which is defined in yaml snippet displayed below and a few pointers are

- app: mssql-sample matches to the selector defined in the service.
- I have specified replicas: 1 which means that only one instance of Pod will be created by Kubernetes for    this deployment. You can update this value as needed.
- The docker image being used to create this deployment is image: microsoft/mssql-server-linux.
- The secret defined in previous step is used in this deployment file to specific sa user password i.e.     - secretKeyRef: name: mssql-sample-secret. Please note that there are SQL server password policy
- Lastly, persistent volume claim created above is also used in this deployment file i.e.         persistentVolumeClaim: claimName: mssql-sample-data-claim.

`sqlserver.dep.yaml`

## Create a Kubernetes Service for core app
The next step is to create a Kubernetes Service for this Web API. As you can see in yaml snippet below since  type: LoadBalancer, AKS is going to create a external endpoint/load balancer ingress for this service.

The creation of this service is going to take a while and once done you can get the external endpoint of this service either by opening AKS Dashboard or running Kubectl command kubectl describe services samplewebapp

`coreapp.svc.yaml`

## Create a Kubernetes Deployment for core app
The next step is to create a Kubernetes Deployment for Angular application. The yaml snippet is displayed below and a few pointers are

You need to update image path i.e. image: atverma/sampleangularapp with your docker hub repository
Two pods will be created for this deployment. You can change the number of pods by updating replicas: 2
Label app: sampleangularapp has to match the selector defined in the service

`coreapp.dep.yaml`
