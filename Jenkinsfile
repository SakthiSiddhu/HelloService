pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_credentials')
        GIT_REPO = "https://github.com/DatlaBharath/HelloService"
        GIT_BRANCH = "main"
        CONTAINER_PORT = 5000
        NODE_PORT = 30007
        K8S_SERVER = "43.205.198.61"
        PEM_FILE = "/var/test.pem"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: env.GIT_BRANCH, url: env.GIT_REPO
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
                    def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                    sh "docker tag ${imageName} ratneshpuskar/helloservice:latest"
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh """
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                            docker push ${imageName}
                            docker push ratneshpuskar/helloservice:latest
                        """
                    }
                }
            }
        }
        
        stage('Create Deployment YAML') {
            steps {
                script {
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
        image: ratneshpuskar/helloservice:${env.BUILD_NUMBER}
        ports:
        - containerPort: ${env.CONTAINER_PORT}
"""
                }
            }
        }
        
        stage('Create Service YAML') {
            steps {
                script {
                    writeFile file: 'service.yaml', text: """
apiVersion: v1
kind: Service
metadata:
  name: helloservice-service
spec:
  selector:
    app: helloservice
  ports:
    - protocol: TCP
      port: ${env.CONTAINER_PORT}
      targetPort: ${env.CONTAINER_PORT}
      nodePort: ${env.NODE_PORT}
  type: NodePort
"""
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sshagent (credentials: ['k8s_ssh_credentials']) {
                        sh """ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@43.205.198.61 "kubectl apply -f -" < deployment.yaml"""
                        sh """ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@43.205.198.61 "kubectl apply -f -" < service.yaml"""
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}