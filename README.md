##Create helm chart anf deploy web application using helm on kubernetes

###Install Helm on your machine

You can follow below link to install helm on your machine
```
https://helm.sh/docs/intro/install/
```

You can follow below link to get used to with helm commands

```
https://helm.sh/docs/intro/using_helm/

helm search hub <CHART_NAME>   //search helm charts on ArtifactHub
helm search repo <CHART_NAME>  //searches the repositories that you have added to your local helm client (with helm repo add)

helm repo add <REPO_NAME> <REPO_URL> //will add repo locally
helm repo ls //to list your local repo

helm install <PROJECT_NAME> <REPO_NAME>/<CHART_NAME>
```

###Create your first helm chart

```
mkdir helm-kubernetes-poc && cd helm-kubernetes-poc
helm create <HELM_CHART_NAME>

Output : Above command would generate some yml files.
ls -R portfolio-app/
portfolio-app/:
Chart.yaml  charts/  templates/  values.yaml

portfolio-app/charts:

portfolio-app/templates:
NOTES.txt  _helpers.tpl  deployment.yaml  hpa.yaml  ingress.yaml  service.yaml  serviceaccount.yaml  tests/

portfolio-app/templates/tests:
test-connection.yaml
```

Now you can take reference from the above generated files to write you own custom helm chart.
Firstly, We will edit the chart.yaml file.

```
cd portfolio-app && vim chart.yaml

apiVersion: v2
name: portfolio-app
description: Helm chart for deploying my portfolio application on kubernetes.
type: application
version: 0.1.0
appVersion: "1.0.0"
maintainers:
  - email: puneetsharma.ms@gmail.com
    name: puneet1051996

```
1. apiVersion: This denotes the chart API version. v2 is for Helm 3 and v1 is for previous versions.
2. name: Denotes the name of the chart.
3. description: Denotes the description of the helm chart.
4. Type: The chart type can be either ‘application’ or ‘library’. Application charts are what you deploy on Kubernetes. Library charts are re-usable charts that can be used with other charts. A similar concept of libraries in programming.
5. Version: This denotes the chart version.
6. appVersion: This denotes the version number of our application (Nginx).
7. maintainers: Information about the owner of the chart.

####Now we will create our own templates

we will create our own deployment.yaml file but the thing is values are kind of static.
The idea behind using helm chart is to make your yaml files generic so that you can use them in
multiple environments.
That is where your **Go Templates** comes into picture.

To template a value, all you need to do is add the object parameter inside curly braces as shown below. It is called a template directive and the syntax is specific to the Go templating
{{ .Object.Parameter }}
Here, objects are of three types:
1. Release: Every helm chart will be deployed with a release name. If you want to use the release name or access release-related dynamic values inside the template, you can use the release object.
2. Chart: If you want to use any values you mentioned in the chart.yaml, you can use the chart object.
3. Values: All parameters inside values.yaml file can be accessed using the Values object.

```
rm -rf templates/*
cd templates && touch deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Chart.Name}} #portfolio-app
spec:
  selector:
    matchLabels:
      app: portfolio
  replicas: {{.Values.replicaCount}} #1
  template:
    metadata:
      labels:
        app: portfolio
    spec:
      containers:
        - name:  {{.Chart.Name}} #portfolio-web-app
          image: {{.Values.image.repository}}:{{.Values.image.tag}} #portfolio:v1.0
          imagePullPolicy: {{.Values.image.pullPolicy}} #IfNotPresent
          ports:
            - containerPort: 80
            
            
touch service.yaml

apiVersion: v1
kind: Service
metadata:
  name: {{.Chart.Name}}-service #portfolio-app-service
spec:
  type: {{.Values.service.type}} #NodePort
  selector:
    app: portfolio
  ports:
    - port: {{.Values.service.port}} #80
      targetPort: {{.Values.service.targetPort}} #8080
```

####Validate Helm Chart

```
helm lint <PATH TO HELM CHART>
```

If there is no error, you will see below result.

```
==> Linting .
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

You can also validate by running **--dry-run** command to check if everything looks good oor not.

```
helm install --dry-run my-portfolio-release portfolio-app
```

####Install and Uninstall helm chart

```
helm install web-app portfolio-app

helm uninstall web-app
```

####To check release list

```
helm list
```