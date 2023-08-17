# k8s-task
## Task

We have a [DigitalOcean sample Django application](https://github.com/digitalocean/sample-django). Our goal is to deploy this app with Kubernetes and store the database in RDS.

We will be using a Helm chart to deploy our application. The Helm chart will contain the following components:

- Create a secret to store our sensitive environment variables such as the database password.
- The deployment will:
  - Launch two replicas of the application.
  - Listen on port 8080.
  - Use the Configmap and Secret.
  - Have readiness and liveness probes.
- Create a ClusterIP service.
- Create an Ingress controller for AWS or DO, depending on the provider, which will raise the corresponding LB.
- Use CertManager to obtain a Let's Encrypt SSL certificate for our application.
- Create a HorizontalPodAutoscaler that will scale the replicas from 2 to 4 based on CPU or memory utilization reaching 80%.

## Prerequisites
- It is necessary to create an RDS database and write out the DATABASE_URL data in this form:
postgresql://USERNAME:PASSWORD@DB_HOST:DB_PORT/DATABASE_NAME

  So USERNAME is the value you have given the field Master Username, PASSWORD - Master Password.

  DATABASE_NAME comes from the "Initial Database name" field, you filled during RDS Postgres Instance creation.

  DB_HOST comes from the value of Endpoint on the database details page.

  DB_PORT comes from Port value on the database details page.

- You need to have a DigitalOcean account and create a Kubernetes cluster there.
- Docker image of our app.
- Install kubectl and helm on your local machine.
- Connect to the cluster using [doctl](https://docs.digitalocean.com/reference/doctl/how-to/install/).
- A domain name and DNS A records which you can point to the DigitalOcean Load Balancer used by the Ingress. If you are using DigitalOcean to manage your domain’s DNS records, consult [How to Manage DNS Records](https://www.digitalocean.com/docs/networking/dns/how-to/manage-records/) to learn how to create A records.


## How to run
### STEP 1 - Download prerequisites

Clone the repository to your computer
```
git clone https://github.com/VitaliySynytskyi/k8s-task
```

After creating RDS, you must get DATABASE_URL and base64 encode it. Then, paste the result in django-chart/templates/secret.yml on line 8 instead of mine.

### STEP 2 - Start metric-server and cert-manager

Run these commands:
```
kubectl apply -f metric-server.yaml
```
```
kubectl apply -f cert-manager.yml
```

### STEP 3 - Run the Helm chart

Run the command:
```
helm install django ./django-chart/
```
And you will get an error:
>Error: INSTALLATION FAILED: failed pre-install: warning: Hook pre-install django-chart/templates/lb-ingress-nginx.yml failed: 1 error occurred:
>       * namespaces "ingress-nginx" not found

Then run the command:
```
kubectl create namespace ingress-nginx
```

After that, run this command:
```
helm upgrade django ./django-chart/
```

You will get the output:

~~~
NAME: django
LAST DEPLOYED: Fri May 12 00:23:09 2023
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
~~~

### STEP 4 - Create DNS Records

Go to the LB tab and wait until it is completely created.

After that, you need to go to the network tab in DO and add the domain name that we prepared earlier and add records that direct to our LB.

It should look something like this:
![image](https://github.com/VitaliySynytskyi/k8s-task/assets/91220971/dff22ec7-a8ea-424b-84dc-4d71786b7254)

After a few minutes, you can access the site with the DNS name by pasting it into your browser!
