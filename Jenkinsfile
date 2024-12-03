pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
            }
        }
        stage('Build Maven project') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Build Docker image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        stage('Push Docker image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    script {
                        sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin'
                        def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                        sh "docker push ${imageName}"
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
  - protocol: TCP
    port: 5000
    targetPort: 5000
    nodePort: 30007
"""
                    def kubernetesServer = "ubuntu@<Kubernetes-Server-IP>"

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh """
ssh -i /var/test.pem -o StrictHostKeyChecking=no ${kubernetesServer} "kubectl apply -f -" < deployment.yaml
ssh -i /var/test.pem -o StrictHostKeyChecking=no ${kubernetesServer} "kubectl apply -f -" < service.yaml
"""
                }
            }
        }
    }
    post {
        success {
            echo 'Build, push and deployment completed successfully.'
        }
        failure {
            echo 'Build, push or deployment failed.'
        }
    }
}