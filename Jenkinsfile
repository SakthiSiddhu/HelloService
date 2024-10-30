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
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def projectName = 'HelloService'.toLowerCase()
                    def dockerTag = 'latest'
                    sh "docker build -t ratneshpuskar/${projectName}:${dockerTag} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                            docker push ratneshpuskar/helloservice:latest
                        '''
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYaml = '''
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
                    def serviceYaml = '''
apiVersion: v1
kind: Service
metadata:
  name: helloservice-service
spec:
  selector:
    app: helloservice
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
'''
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
}