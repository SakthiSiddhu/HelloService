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

        stage('Build Package') {
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
            environment {
                DOCKERHUB_CREDENTIALS = credentials('dockerhub_credentials')
            }
            steps {
                script {
                    def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh "echo \"$DOCKERHUB_CREDENTIALS_PSW\" | docker login -u \"$DOCKERHUB_CREDENTIALS_USR\" --password-stdin"
                    sh "docker push ${imageName}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
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
                            image: ${imageName}
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
                          nodePort: 30007
                      type: NodePort
                    """
                    sh "echo '${deploymentYaml}' > deployment.yaml"
                    sh "echo '${serviceYaml}' > service.yaml"
                    sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@15.207.87.59 "sudo lsof -t -i :30007 | xargs -r sudo kill -9" || true
                    """
                    sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@15.207.87.59 "kubectl apply -f -" < deployment.yaml
                    """
                    sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@15.207.87.59 "kubectl apply -f -" < service.yaml
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build, Push, and Deployment completed successfully.'
        }
        failure {
            echo 'An error occurred during build, push, or deployment.'
        }
    }
}