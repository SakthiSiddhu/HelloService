pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        GIT_REPO = 'https://github.com/DatlaBharath/HelloService'
        BRANCH = 'main'
        DOCKER_IMAGE = 'ratneshpuskar/helloservice'
        DOCKER_CREDENTIALS_ID = 'dockerhub_credentials'
        SSH_KEY_PATH = '/var/test.pem'
        SSH_USER = 'ec2-user'
        SSH_HOST = '13.235.128.206'
        CONTAINER_PORT = '5000'
        SERVICE_NAME = 'helloservice'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: env.BRANCH, url: env.GIT_REPO
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def tag = 'latest'
                    sh "docker build -t ${env.DOCKER_IMAGE}:${tag} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh """
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${env.DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    writeFile file: 'deployment.yaml', text: """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${env.SERVICE_NAME}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${env.SERVICE_NAME}
  template:
    metadata:
      labels:
        app: ${env.SERVICE_NAME}
    spec:
      containers:
      - name: ${env.SERVICE_NAME}
        image: ${env.DOCKER_IMAGE}:latest
        ports:
        - containerPort: ${env.CONTAINER_PORT}
"""

                    writeFile file: 'service.yaml', text: """
apiVersion: v1
kind: Service
metadata:
  name: ${env.SERVICE_NAME}
spec:
  selector:
    app: ${env.SERVICE_NAME}
  ports:
    - protocol: TCP
      port: ${env.CONTAINER_PORT}
      targetPort: ${env.CONTAINER_PORT}
"""

                    sh """
                        ssh -i ${env.SSH_KEY_PATH} -o StrictHostKeyChecking=no ${env.SSH_USER}@${env.SSH_HOST} "kubectl apply -f -" < deployment.yaml
                        ssh -i ${env.SSH_KEY_PATH} -o StrictHostKeyChecking=no ${env.SSH_USER}@${env.SSH_HOST} "kubectl apply -f -" < service.yaml
                        sleep 60
                        ssh -i ${env.SSH_KEY_PATH} -o StrictHostKeyChecking=no ${env.SSH_USER}@${env.SSH_HOST} sudo kill -9 /$(sudo lsof -t -i :${env.CONTAINER_PORT})
                        ssh -i ${env.SSH_KEY_PATH} -o StrictHostKeyChecking=no ${env.SSH_USER}@${env.SSH_HOST} "kubectl port-forward --address 0.0.0.0 service/${env.SERVICE_NAME} ${env.CONTAINER_PORT}:${env.CONTAINER_PORT}"
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}