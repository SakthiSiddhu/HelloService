
pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_credentials')
        REPO_URL = 'https://github.com/DatlaBharath/HelloService'
        BRANCH = 'main'
        IMAGE_NAME = 'ratneshpuskar/helloservice'
        CLUSTER_CREDENTIALS = '/var/test.pem'
        CLUSTER_USER = 'ec2-user'
        CLUSTER_HOST = '15.206.72.215'
        K8S_PORT = '5000'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${IMAGE_NAME}:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageTag} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def imageTag = "${IMAGE_NAME}:${env.BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: '${DOCKERHUB_CREDENTIALS}', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        sh '''
                            echo $PASSWORD | docker login -u $USERNAME --password-stdin
                            docker push ${imageTag}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def imageTag = "${IMAGE_NAME}:${env.BUILD_NUMBER}"
                    writeFile file: 'deployment.yaml', text: """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloservice-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloservice
  template:
    metadata:
      labels:
        app: helloservice
    spec:
      containers:
      - name: helloservice
        image: ${imageTag}
        ports:
        - containerPort: ${K8S_PORT}
"""
                    writeFile file: 'service.yaml', text: """
apiVersion: v1
kind: Service
metadata:
  name: helloservice-service
spec:
  ports:
  - port: ${K8S_PORT}
    targetPort: ${K8S_PORT}
  selector:
    app: helloservice
"""

                    sh """
                        ssh -i ${CLUSTER_CREDENTIALS} -o StrictHostKeyChecking=no ${CLUSTER_USER}@${CLUSTER_HOST} "kubectl apply -f -" < deployment.yaml
                        ssh -i ${CLUSTER_CREDENTIALS} -o StrictHostKeyChecking=no ${CLUSTER_USER}@${CLUSTER_HOST} "kubectl apply -f -" < service.yaml
                        sleep 60
                        ssh -i ${CLUSTER_CREDENTIALS} -o StrictHostKeyChecking=no ${CLUSTER_USER}@${CLUSTER_HOST} "sudo lsof -t -i :${K8S_PORT} | xargs -r sudo kill -9" || true
                        ssh -i ${CLUSTER_CREDENTIALS} -o StrictHostKeyChecking=no ${CLUSTER_USER}@${CLUSTER_HOST} "kubectl port-forward --address 0.0.0.0 service/helloservice-service ${K8S_PORT}:${K8S_PORT}" || true
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}