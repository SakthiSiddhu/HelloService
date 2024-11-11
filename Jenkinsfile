pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/DatlaBharath/HelloService']]])
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    def dockerImage = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh "docker build -t ${dockerImage} ."
                }
            }
        }

        stage('Docker Login and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh 'docker login -u ${USER} -p ${PASS}'
                    sh "docker push ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deployment = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloservice
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

                    def service = """
apiVersion: v1
kind: Service
metadata:
  name: helloservice
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
                    writeFile file: 'deployment.yaml', text: deployment
                    writeFile file: 'service.yaml', text: service

                    sh 'scp -i /var/test.pem -o StrictHostKeyChecking=no deployment.yaml ubuntu@13.233.247.3:/home/ubuntu/'
                    sh 'scp -i /var/test.pem -o StrictHostKeyChecking=no service.yaml ubuntu@13.233.247.3:/home/ubuntu/'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.247.3 "kubectl apply -f /home/ubuntu/deployment.yaml"'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.247.3 "kubectl apply -f /home/ubuntu/service.yaml"'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
