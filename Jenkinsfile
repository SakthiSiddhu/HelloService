pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
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
                    def projectName = 'helloservice'.toLowerCase()
                    def dockerTag = 'latest'
                    def dockerImage = "ratneshpuskar/${projectName}:${dockerTag}"

                    sh "docker build -t ${dockerImage} ."

                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                        sh "docker push ${dockerImage}"
                    }
                }
            }
        }
        stage('Kubernetes Deploy') {
            steps {
                writeFile file: 'deployment.yaml', text: '''
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
        image: ratneshpuskar/helloservice:latest
        ports:
        - containerPort: 5000
'''
                writeFile file: 'service.yaml', text: '''
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
      nodePort: 30000
'''
                sh '''
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < service.yaml

                    sleep 60

                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true

                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000"
                '''
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