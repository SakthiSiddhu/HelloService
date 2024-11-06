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
        stage('Build Docker Image') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    def dockerImage = "ratneshpuskar/${projectName}:${dockerTag}"
                    
                    sh 'docker build -t ${dockerImage} .'
                    
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                        sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                        sh 'docker push ${dockerImage}'
                    }
                }
            }
        }
        stage('Create Deployment Files') {
            steps {
                script {
                    def deploymentYaml = '''apiVersion: apps/v1
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
        - containerPort: 5000'''
                    
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    
                    def serviceYaml = '''apiVersion: v1
kind: Service
metadata:
  name: helloservice
spec:
  selector:
    app: helloservice
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000'''
                    
                    writeFile file: 'service.yaml', text: serviceYaml
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@43.205.240.201 "kubectl apply -f -" < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@43.205.240.201 "kubectl apply -f -" < service.yaml
                    sleep 60
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@43.205.240.201 "kubectl port-forward --address 0.0.0.0 service/helloservice 5000:80" 
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}