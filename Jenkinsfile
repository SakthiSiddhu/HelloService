pipeline {
    agent any

    environment {
        KUBECONFIG = "/home/ec2-user/.kube/config"
    }

    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
            }
        }
        
        stage('Build Maven Project') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerTag = "v1.0.0"
                    def project = "helloservice"
                    sh """
                        docker build -t ratneshpuskar/${project.toLowerCase()}:${dockerTag} .
                    """
                }
            }
        }
        
        stage('Docker Login & Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh """
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push ratneshpuskar/helloservice:v1.0.0
                    """
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    writeFile file: 'deployment.yaml', text: """
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: helloservice
                      labels:
                        app: helloservice
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
                            image: ratneshpuskar/helloservice:v1.0.0
                            ports:
                            - containerPort: 5000
                    """

                    writeFile file: 'service.yaml', text: """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: helloservice
                    spec:
                      type: LoadBalancer
                      ports:
                      - port: 5000
                        targetPort: 5000
                      selector:
                        app: helloservice
                    """

                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
}