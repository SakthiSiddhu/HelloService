pipeline {
    agent any
    tools {
        maven 'Maven_3.9.9'
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
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerTag = "${env.BUILD_NUMBER}"
                    def imageName = 'bharathkamal/helloservice:' + dockerTag
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        stage('Docker Login and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'docker push ${imageName}'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deployment = '''
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
        image: ${imageName}
        ports:
        - containerPort: 5000
'''
                    def service = '''
apiVersion: v1
kind: Service
metadata:
  name: helloservice
spec:
  type: NodePort
  selector:
    app: helloservice
  ports:
    - port: 5000
      targetPort: 5000
      nodePort: 30007
'''
                    writeFile file: 'deployment.yaml', text: deployment
                    writeFile file: 'service.yaml', text: service
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
}
