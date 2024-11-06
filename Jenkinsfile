pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        REPO_NAME = 'helloservice'
        REPO_URL = 'https://github.com/DatlaBharath/HelloService'
        BRANCH_NAME = 'main'
        DOCKER_IMAGE = 'ratneshpuskar/helloservice:dockertag'
        DOCKER_CREDENTIALS_ID = 'dockerhub_credentials'
        DEPLOYMENT_USER = 'ec2-user'
        DEPLOYMENT_SERVER = '13.235.128.206'
        DEPLOYMENT_KEY = '/var/test.pem'
        CONTAINER_PORT = 5000
        KUBERNETES_SERVICE = 'helloservice-service'
        SERVICE_PORT = 5000
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${BRANCH_NAME}", url: "${REPO_URL}"
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def deploymentYaml = """apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${REPO_NAME}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${REPO_NAME}
  template:
    metadata:
      labels:
        app: ${REPO_NAME}
    spec:
      containers:
      - name: ${REPO_NAME}
        image: ${DOCKER_IMAGE}
        ports:
        - containerPort: ${CONTAINER_PORT}
"""
                    def serviceYaml = """apiVersion: v1
kind: Service
metadata:
  name: ${KUBERNETES_SERVICE}
spec:
  selector:
    app: ${REPO_NAME}
  ports:
  - protocol: TCP
    port: ${SERVICE_PORT}
    targetPort: ${CONTAINER_PORT}
"""
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml
                }

                sh """
                    ssh -i ${DEPLOYMENT_KEY} -o StrictHostKeyChecking=no ${DEPLOYMENT_USER}@${DEPLOYMENT_SERVER} "kubectl apply -f -" < deployment.yaml
                    ssh -i ${DEPLOYMENT_KEY} -o StrictHostKeyChecking=no ${DEPLOYMENT_USER}@${DEPLOYMENT_SERVER} "kubectl apply -f -" < service.yaml
                    sleep 60
                    ssh -i ${DEPLOYMENT_KEY} -o StrictHostKeyChecking=no ${DEPLOYMENT_USER}@${DEPLOYMENT_SERVER} "sudo lsof -t -i :${CONTAINER_PORT} | xargs -r sudo kill -9" || true
                    ssh -i ${DEPLOYMENT_KEY} -o StrictHostKeyChecking=no ${DEPLOYMENT_USER}@${DEPLOYMENT_SERVER} "kubectl port-forward --address 0.0.0.0 service/${KUBERNETES_SERVICE} ${CONTAINER_PORT}:${SERVICE_PORT}"
                """
            }
        }
    }

    post {
        success {
            echo 'Deployment successful.'
        }

        failure {
            echo 'Deployment failed.'
        }
    }
}