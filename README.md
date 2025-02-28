# DevOps Project

## Contributors - Team Z
- Esteban VINCENT
- Zyad ZEKRI
- Mehdi AMMOR

## Subject description
The DevOps project aims to establish a complete **CI/CD pipeline** for a demo **Go application**. This application was used to showcase the automated deployment process and implement modern DevOps practices.

The **CI/CD pipeline** was built using **Jenkins**, incorporating steps like compilation, testing, Docker image creation, and automated deployment to a Kubernetes cluster. The infrastructure is designed to demonstrate how to automate the software development lifecycle.

Once deployed, the project also includes setting up a **monitoring system** to track the health of the application and Kubernetes cluster resources. This was achieved with the following tools:
- **Kubernetes** for managing deployments and services,
- **Prometheus** for collecting metrics,
- **Grafana** for visualizing data,
- **AlertManager** for managing alerts and notifying teams in case of issues.

The ultimate goal of this project is to demonstrate a full DevOps solution with continuous integration, real-time monitoring, and alert management within a Kubernetes environment.

This project has been completed on Ubuntu runing in Windows 11 WSL.

## Technical description

### Prerequisites
You'll require those:
- A running Kubernetes cluster
- Jenkins
- A Docker Hub account
- A mail account with the password for apps
- Docker installed on your machine

### Part 1 - CI / CD
#### Accessing Jenkins
1. Launch Jenkins
2. Read the instructions to get the password and copy it
3. Access Jenkins at http://localhost:8080
4. Login to Jenkins with `admin` as username and the password you just copied

#### Create the first pipeline
1. Click on `New Item` button
2. Enter a name for the pipeline, and select `Pipeline`
3. Click on `OK` button
4. In the Pipeline Definition field, select `Pipeline Script from SCM`
5. On the SCM field, select `Git`, and then set the Repository URL to `https://github.com/BadTimeGiver/DevOps-Project-Team-Z`
6. Set the Branch Specifier field to `main`
7. Click on the `Save` button

#### Set the Credentials of the DockerHub
1. Go on `http://localhost:8080/manage/credentials/store/system/domain/_/`
2. Click on the `Add Credentials` button
3. Set your DockerHub username in the `Username` field
4. Set your DockerHub password in the `Password` field
5. Set `dockerhub-creds` in the `ID` field

#### Set the plugin
1. Go on `http://localhost:8080/manage/pluginManager/`
2. Install the Git plugin
3. Install the Docker plugin

#### Launch the pipeline
1. In the main Dashboard, select the pipeline you have created for this project
2. Click on `Launch a Build` button

### Part 2 - Monitoring
#### Installing & Deploying Prometheus & Grafana
1. Add the following Helm repositories:
```
helm repo add -n default prometheus-community https://prometheus-community
helm repo add -n default grafana https://grafana.github.io/helm-charts
helm repo update
```

2. Install Prometheus and Grafana
```
helm install -n default prometheus prometheus-community/prometheus
helm install -n default grafana grafana/grafana
```

3. Get the Prometheus server URL by running the following command
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace default port-forward $POD_NAME 9090
```

4. Get the Grafana password by running the following command
```
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

5. Get the Grafana server URL by running the following command
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=grafana" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace default port-forward $POD_NAME 3000
```

6. Access Prometheus at http://localhost:9090 and Grafana at http://localhost:3000 (and login to Grafana with the password from 4 and admin for the username)

#### Configuring Grafana
1. Add Prometheus as a Data Source in Grafana
    - Go on your Grafana web page
    - Click on Connections > Data Sources on the sidebar
    - Click on Add Data Source
    - Select Prometheus for the data source
    - Set the URL to `http://prometheus-server.default.svc.cluster.local`
    - Click `Save & Test`

2. Import the dashboard
    - Click on the `Dashboards` button on the sidebar
    - Click on the `Create dashboard` button
    - Click on the `Import dashboard` button
    - Enter the dashboard ID `1860` in the `Grafana.com dashboard` field
    - Click on the `Load` button just next to this field
    - Select the Prometheus data source
    - Click on the `Load` button

#### Configuring Alert Manager & Prometheus for alert management
1. Edit the file `AlertManager/AlertManagerConfiguration.yaml` with your credentials

2. To be able to access the Alert Mananger from your browser, run those commands
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=alertmanager,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9093
```

3. Run those commands to setup Alert Manager and the Prometheus alert
```
helm upgrade --reuse-values -f AlertManager/alert.yaml prometheus prometheus-community/prometheus
helm upgrade --reuse-values -f AlertManager/AlertManagerConfiguration.yaml prometheus
```

4. If you wanna test the alert, you can run the following command:
```
kubectl delete deployment prometheus-prometheus-pushgateway
```
You'll then receive an email

5. Access on the `localhost:9090/alerts` to see the list of alerts

### Part 3 - Logs Management