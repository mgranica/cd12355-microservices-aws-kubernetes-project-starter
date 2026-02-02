# Coworking Space Service Extension
The Coworking Space Service is a set of APIs that enables users to request one-time tokens and administrators to authorize access to a coworking space. This service follows a microservice pattern and the APIs are split into distinct services that can be deployed and managed independently of one another.

For this project, you are a DevOps engineer who will be collaborating with a team that is building an API for business analysts. The API provides business analysts basic analytics data on user activity in the service. The application they provide you functions as expected locally and you are expected to help build a pipeline to deploy it in Kubernetes.

## Getting Started

### Dependencies
#### Remote Resources
1. AWS CodeBuild - build Docker images remotely
2. AWS ECR - host Docker images
3. Kubernetes Environment with AWS EKS - run applications in k8s
4. AWS CloudWatch - monitor activity and logs in EKS
5. GitHub - pull and clone code

## Cluster Usage

### Prerequisites

* Ensure the AWS CLI is configured correctly.

```
aws sts get-caller-identity
```


### Step 1. Create EKS cluster

* Setup the Kubernetes cluster, If not enabled yet:
```
eksctl create cluster --name coworking --region us-east-1 --nodegroup-name my-nodes --node-type t3.small --nodes 1 --nodes-min 1 --nodes-max 2

```

### Step 2.  Update The kubeconfig
* Update the context in the local `Kubeconfig`file to access the cluster

```
aws eks --region us-east-1 update-kubeconfig --name coworking
```

* Verify context
````
kubectl config current-context
````

### Step 3. Configure Services

#### Step 3.1 Configure Database
* Apply YAML configuration for DB service
```
kubectl apply -f pvc.yaml
kubectl apply -f pv.yaml
kubectl apply -f postgresql-deployment.yaml
kubectl apply -f postgresql-service.yaml
```

#### 3.2 Configure Analytics Application

* Apply YAML configuration for Analytics service
* Default Variables and secrets are set. For any modification contact EKS owner.
```
kubectl apply -f deployment/configmap.yaml
kubectl apply -f secrets.yaml
kubectl apply -f deployment/coworking.yaml
```

* Check App availability
```
kubectl get pods
```

```
kubectl get svc
```

### Step 4. Query analytics API

* BASE URL = `a75b7d325d4bc49fc9e346937590e489-1844286969.us-east-1.elb.amazonaws.com:5153`

  * users visits

```
curl <BASE_URL>:5153/api/reports/user_visits
``` 
  * daily usage
```
curl <BASE_URL>:5153/api/reports/daily_usage
```

### 5. Seed Data
* In case it is needed you can reseed the data with the following commands

```
kubectl port-forward --address 127.0.0.1 service/postgresql-service 5433:5432 &
export DB_PASSWORD=`kubectl get secret project3-secrets -o jsonpath='{.data.password}' | base64 --decode`
export DB_USER=`kubectl get configMap project3-config-map -o jsonpath='{.data.DB_USER}'`
export DB_NAME=`kubectl get configMap project3-config-map -o jsonpath='{.data.DB_NAME}'`
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U ${DB_USER} -d ${DB_NAME} -p 5433 < ./db/1_create_tables.sql
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U ${DB_USER} -d ${DB_NAME} -p 5433 < ./db/2_seed_users.sql
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U ${DB_USER} -d ${DB_NAME} -p 5433 < ./db/3_seed_tokens.sql
```

### 6. Logging
* CloudWatch had policy granted to the cluster logs,
```
aws iam attach-role-policy \
    --role-name eksctl-coworking-nodegroup-my-node-NodeInstanceRole-2Yog6gLdYTjd \
    --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy
```

* Attach the service policy
```aws eks create-addon --addon-name amazon-cloudwatch-observability --cluster-name coworking```