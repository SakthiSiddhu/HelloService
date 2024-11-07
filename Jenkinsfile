pipeline {
    agent any
    
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
                    def projectName = 'helloservice'
                    def dockerTag = 'dockertag'
                    docker.build("ratneshpuskar/${projectName}:${dockerTag}")
                }
            }
        }
        
        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'dockertag'
                    sh "docker push ratneshpuskar/${projectName}:${dockerTag}"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYaml = """
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
                            image: ratneshpuskar/helloservice:dockertag
                            ports:
                            - containerPort: 5000
                    """
                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: helloservice-service
                    spec:
                      selector:
                        app: helloservice
                      ports:
                        - protocol: TCP
                          port: 5000
                          targetPort: 5000
                    """
                    
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml
                    
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "kubectl apply -f -" < service.yaml'
                    sleep 60
                    
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000" || true'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Job completed successfully'
        }
        failure {
            echo 'Job failed'
        }
    }
}