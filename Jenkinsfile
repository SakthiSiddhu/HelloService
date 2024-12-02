pipeline {
    agent any
    tools {
        jdk 'java-11'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/username/reponame.git']]])
            }
        }
        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def repoName = 'reponame'.toLowerCase()
                    def imageTag = "ratneshpuskar/${repoName}:${env.BUILD_NUMBER}"
                    
                    sh "docker build -t ${imageTag} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    sh """
                        echo "${DOCKER_HUB_PASSWORD}" | docker login -u "${DOCKER_HUB_USERNAME}" --password-stdin
                        docker push ratneshpuskar/${repoName}:${env.BUILD_NUMBER}
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
  name: ${repoName}-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${repoName}
  template:
    metadata:
      labels:
        app: ${repoName}
    spec:
      containers:
      - name: ${repoName}
        image: ratneshpuskar/${repoName}:${env.BUILD_NUMBER}
        ports:
        - containerPort: 8080
"""

                    def serviceYaml = """
apiVersion: v1
kind: Service
metadata:
  name: ${repoName}-service
spec:
  type: NodePort
  selector:
    app: ${repoName}
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30007
"""

                    sh """
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.107.58.12 "kubectl apply -f -" <<< "${deploymentYaml}"
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.107.58.12 "kubectl apply -f -" <<< "${serviceYaml}"
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment succeeded'
        }
        failure {
            echo 'Deployment failed'
        }
    }
}