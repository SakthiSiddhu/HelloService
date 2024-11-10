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
        
        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageTag} ."
                    
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh "docker push ${imageTag}"
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
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
        image: ratneshpuskar/helloservice:${env.BUILD_NUMBER}
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
  - nodePort: 30007
    port: 5000
    targetPort: 5000
  type: NodePort
                '''
                
                sh ''' 
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@43.205.198.61 'kubectl apply -f -' < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@43.205.198.61 'kubectl apply -f -' < service.yaml
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Build and deployment succeeded!'
        }
        
        failure {
            echo 'Build or deployment failed.'
        }
    }
}