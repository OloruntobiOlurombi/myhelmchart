# Jenkins on Kubernetes with Helm

This repository provides a detailed guide and resources for deploying Jenkins on a Kubernetes cluster using Helm. This project aims to simplify the Jenkins deployment process and enhance your understanding of both Helm and Kubernetes.


## Table of Contents
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Deployment Instructions](#deployment-instructions)
- [Accessing Jenkins](#accessing-jenkins)
- [Contributing](#contributing)

## Features
- Step-by-step instructions for creating a custom Helm chart for Jenkins.
- Configurable Kubernetes deployments and services.
- Easy access to Jenkins via a web browser using the node's public IP.
- Template files for Helm charts included for your convenience.

## Prerequisites
Before you begin, ensure you have the following installed:

- Kubernetes (e.g., Minikube, kubeadm, or a cloud provider)

- Helm

- Access to a terminal with `kubectl` and `helm` commands available

## Deployment Instructions

1. Create a custom Helm chart:

```bash
helm create myhelmchart
```

2. Update `Chart.yaml` to reflect the Jenkins version you are using:

```bash
appVersion: "2.249.2"
```

3. Remove unwanted files and directories from the Helm chart:

```bash
rm -rf myhelmchart/templates/*.yaml myhelmchart/templates/tests
```

4. Create the Deployment and Service templates in the `templates` directory:

- Create `jenkins-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: {{ template "myhelmchart.fullname" . }}
  labels:
    app: jenkins 
spec:
  replicas: 1
  selector: 
    matchLabels:
      app: jenkins 
  template:
    metadata:
      labels:
        app: jenkins 
    spec:
      containers:
        - name: jenkins
          image: {{ printf "%s:%s" .Values.image.repository .Values.image.tag | quote }}
          env: 
            - name: JENKINS_USERNAME 
              value: {{ default "test" .Values.jenkinsUsername | quote }}
            - name: JENKINS_PASSWORD
              {{- if .Values.jenkinsPassword }}
              value: {{.Values.jenkinsPassword }}
              {{- else }}
              value: testPassword 
              {{- end }}
          ports:
            {{- range .Values.containerPorts }} 
            - name: {{ .name }}
              containerPort: {{ .port }}
            {{- end }}
```

- Create `jenkins-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ template "myhelmchart.fullname" . }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - name: http
      port: 8080
      targetPort: http
      nodePort: {{ .Values.service.nodePort }}
  selector:
    app: jenkins
```

5. Update `values.yaml` with the following configuration:

```yaml
image: 
  repository: bitnami/jenkins 
  tag: latest 

jenkinsUsername: ""
jenkinsPassword: ""

containerPorts:
  - name: http 
    port: 8080 

service:
  type: NodePort
  nodePort: 30080
```

6. Run the Helm template command to render the Kubernetes resources:

```bash
helm template myhelmchart . -s templates/jenkins-deployment.yaml
```

7. Install the Helm chart:

```bash 
helm install myhelmchart ./myhelmchart
```

## Accessing Jenkins

Once Jenkins is deployed, you can access it using the public IP of the node (EC2 instance). Open your browser and navigate to:

```vbnet
http://<node-public-ip>:30080
```

Use the default credentials to log in:

- ***Username***: test

- ***Password***: testPassword

### Contributing

Contributions are welcome! If you have suggestions for improvements or find any issues, please open an issue or submit a pull request.