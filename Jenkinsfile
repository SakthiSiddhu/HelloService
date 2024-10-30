pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        KUBECONFIG = '/home/ec2-user/.kube/config'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            environment {
                PROJECT_NAME = 'helloservice'
                DOCKER_TAG = 'latest'
            }
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_REGISTRY}/${PROJECT_NAME}:${DOCKER_TAG}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                    sh 'docker push ${DOCKER_REGISTRY}/${PROJECT_NAME}:${DOCKER_TAG}'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    writeFile file: 'deployment.yml', text: """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${PROJECT_NAME}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${PROJECT_NAME}
  template:
    metadata:
      labels:
        app: ${PROJECT_NAME}
    spec:
      containers:
      - name: ${PROJECT_NAME}
        image: ${DOCKER_REGISTRY}/${PROJECT_NAME}:${DOCKER_TAG}
        ports:
        - containerPort: 5000
"""
                    writeFile file: 'service.yml', text: """
apiVersion: v1
kind: Service
metadata:
  name: ${PROJECT_NAME}
spec:
  selector:
    app: ${PROJECT_NAME}
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
"""
                    sh 'kubectl apply -f deployment.yml'
                    sh 'kubectl apply -f service.yml'
                }
            }
        }
    }
}