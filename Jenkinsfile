pipeline {
    agent any 
    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService'
            }
        }
        
        stage('Build Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    sh "echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USERNAME --password-stdin"
                    sh "docker push ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                writeFile file: 'deployment.yaml', text: """
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
                """

                writeFile file: 'service.yaml', text: """
                apiVersion: v1
                kind: Service
                metadata:
                  name: helloservice-service
                spec:
                  type: NodePort
                  ports:
                  - port: 5000
                    nodePort: 30007
                    targetPort: 5000
                  selector:
                    app: helloservice
                """

                sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.107.58.12 "kubectl apply -f -" < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.107.58.12 "kubectl apply -f -" < service.yaml
                """
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