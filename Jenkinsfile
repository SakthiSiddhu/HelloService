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
                    def imageName = "ratneshpuskar/${env.JOB_NAME.toLowerCase()}:${env.BUILD_NUMBER}"
                    sh 'docker build -t ${imageName} .'
                }
            }
        }
        
        stage('Docker Login and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'docker push ${imageName}'
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                writeFile file: 'deployment.yaml', text: '''
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
                writeFile file: 'service.yaml', text: '''
apiVersion: v1
kind: Service
metadata:
  name: helloservice
spec:
  selector:
    app: helloservice
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
'''
                sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.124.162 "kubectl apply -f -" < deployment.yaml'
                sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.124.162 "kubectl apply -f -" < service.yaml'
            }
        }

        stage('Wait for Deployment') {
            steps {
                script {
                    sleep 60
                }
            }
        }
        
        stage('Port Forward Kubernetes Service') {
            steps {
                sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.124.162 kubectl port-forward --address 0.0.0.0 service/helloservice 5000:5000'
            }
        }
    }
    
    post {
        success {
           echo 'Deployment successful.'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}