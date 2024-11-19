pipeline {
    agent any
    tools {
        maven 'Maven'
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
        stage('Docker Build and Push') {
            steps {
                script {
                    def dockerImage = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                        sh """echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin"""
                        sh "docker build -t ${dockerImage} ."
                        sh "docker push ${dockerImage}"
                    }
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
        - containerPort: 5000
"""
                    def serviceYaml = """
apiVersion: v1
kind: Service
metadata:
  name: helloservice-service
spec:
  type: NodePort
  selector:
    app: helloservice
  ports:
    - port: 5000
      targetPort: 5000
      nodePort: 30007
"""
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml
                    sh 'chmod 600 /var/test.pem'
                    sh """ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@3.110.120.24 "kubectl apply -f -" < deployment.yaml"""
                    sh """ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@3.110.120.24 "kubectl apply -f -" < service.yaml"""
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