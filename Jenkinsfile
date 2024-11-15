pipeline {
    agent any

    tools { 
        maven "Maven" 
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "https://github.com/DatlaBharath/HelloService", branch: "main"
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def repoName = "helloservice"
                    def tag = "ratneshpuskar/${repoName.toLowerCase()}:${env.BUILD_NUMBER}"
                    sh "docker build -t ${tag} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    script {
                        def repoName = "helloservice"
                        def tag = "ratneshpuskar/${repoName.toLowerCase()}:${env.BUILD_NUMBER}"
                        sh 'echo "$DOCKER_HUB_PASSWORD" | docker login -u "$DOCKER_HUB_USERNAME" --password-stdin'
                        sh "docker push ${tag}"
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
  ports:
    - port: 5000
      nodePort: 30007
  selector:
    app: helloservice
"""
                    sh """echo '${deploymentYaml}' | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@65.2.9.62 "kubectl apply -f -" """
                    sh """echo '${serviceYaml}' | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@65.2.9.62 "kubectl apply -f -" """
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