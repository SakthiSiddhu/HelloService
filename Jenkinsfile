pipeline {
    agent any
    
    tools {
        maven 'Maven'
    }
    
    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub_credentials'
        DOCKER_IMAGE_NAME = 'ratneshpuskar/helloservice'
        DOCKER_TAG = 'v1.0'
        DEPLOYMENT_YAML = 'deployment.yaml'
        SERVICE_YAML = 'service.yaml'
        PRIVATE_KEY_PATH = '/var/test.pem'
        REMOTE_USER = 'ec2-user'
        REMOTE_HOST = '13.235.128.206'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService'
            }
        }
        
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.DOCKER_IMAGE_NAME}:${env.DOCKER_TAG}")
                }
            }
        }
        
        stage('Docker Login and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${env.DOCKER_CREDENTIALS_ID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'docker login -u $USERNAME -p $PASSWORD'
                    sh 'docker push ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_TAG}'
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                writeFile file: "${env.DEPLOYMENT_YAML}", text: """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloservice-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: helloservice
    spec:
      containers:
      - name: helloservice
        image: ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_TAG}
        ports:
        - containerPort: 5000
  selector:
    matchLabels:
      app: helloservice
                """
                
                writeFile file: "${env.SERVICE_YAML}", text: """
apiVersion: v1
kind: Service
metadata:
  name: helloservice-service
spec:
  selector:
    app: helloservice
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
                """
                
                sh 'ssh -i ${env.PRIVATE_KEY_PATH} -o StrictHostKeyChecking=no ${env.REMOTE_USER}@${env.REMOTE_HOST} "kubectl apply -f -" < ${env.DEPLOYMENT_YAML}'
                sh 'ssh -i ${env.PRIVATE_KEY_PATH} -o StrictHostKeyChecking=no ${env.REMOTE_USER}@${env.REMOTE_HOST} "kubectl apply -f -" < ${env.SERVICE_YAML}'
                sleep 60
                sh 'ssh -i ${env.PRIVATE_KEY_PATH} -o StrictHostKeyChecking=no ${env.REMOTE_USER}@${env.REMOTE_HOST} "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true'
                sh 'ssh -i ${env.PRIVATE_KEY_PATH} -o StrictHostKeyChecking=no ${env.REMOTE_USER}@${env.REMOTE_HOST} "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000"'
            }
        }
    }
    
    post {
        success {
            echo 'Deployment was successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}