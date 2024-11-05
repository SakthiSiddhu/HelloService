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
                    def projectName = 'helloservice'.toLowerCase()
                    def dockerTag = 'latest'
                    def imageName = "ratneshpuskar/${projectName}:${dockerTag}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        stage('Docker Login and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh "docker push ratneshpuskar/helloservice:latest"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
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
  selector:
    app: helloservice
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
                    '''
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.124.162 kubectl apply -f - < deployment.yaml --validate=false'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.124.162 kubectl apply -f - < service.yaml --validate=false'
                }
            }
        }
        stage('Port Forward') {
            steps {
                sleep 60
                script {
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.124.162 kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:80'
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully and completed deployment.'
        }
        failure {
            echo 'Pipeline has failed.'
        }
    }
}