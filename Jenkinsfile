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
        stage('Build with Maven') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Docker Build and Push') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    def dockerImage = "ratneshpuskar/${projectName}:${dockerTag}"
                    
                    sh "docker build -t ${dockerImage} ."
                    
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh "docker push ${dockerImage}"
                    }
                }
            }
        }
        stage('Kubernetes Deployment') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    def dockerImage = "ratneshpuskar/${projectName}:${dockerTag}"
                    
                    sh """cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $projectName-deployment
  labels:
    app: $projectName
spec:
  replicas: 1
  selector:
    matchLabels:
      app: $projectName
  template:
    metadata:
      labels:
        app: $projectName
    spec:
      containers:
      - name: $projectName
        image: $dockerImage
        ports:
        - containerPort: 5000
EOF"""
                    sh """cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: $projectName-service
spec:
  selector:
    app: $projectName
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
EOF"""

                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
}