pipeline {
    agent any
    tools {
        tool ''
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: ' ']], userRemoteConfigs: [[url: ' ']]])
            }
        }
        stage('Build Project') {
            steps {
                sh """
                ./mvnw clean package -DskipTests
                """
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = "ratneshpuskar/$(env.JOB_NAME.toLowerCase()):${env.BUILD_NUMBER}"
                    sh "docker build -t ${dockerImage} ."
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                    echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                    docker push ratneshpuskar/$(env.JOB_NAME.toLowerCase()):${env.BUILD_NUMBER}
                    """
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYaml = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${env.JOB_NAME.toLowerCase()}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${env.JOB_NAME.toLowerCase()}
  template:
    metadata:
      labels:
        app: ${env.JOB_NAME.toLowerCase()}
    spec:
      containers:
      - name: ${env.JOB_NAME.toLowerCase()}
        image: ratneshpuskar/$(env.JOB_NAME.toLowerCase()):${env.BUILD_NUMBER}
        ports:
        - containerPort: 
"""
                    def serviceYaml = """
apiVersion: v1
kind: Service
metadata:
  name: ${env.JOB_NAME.toLowerCase()}-service
spec:
  type: NodePort
  selector:
    app: ${env.JOB_NAME.toLowerCase()}
  ports:
    - protocol: TCP
      port: 
 nodePort: 30007
"""
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml
                    sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@"kubectl apply -f -" < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@"kubectl apply -f -" < service.yaml
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Job succeeded!'
        }
        failure {
            echo 'Job failed!'
        }
    }
}