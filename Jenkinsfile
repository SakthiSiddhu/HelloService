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
                    def appName = "helloservice"
                    def appVersion = "latest"
                    def dockerImage = "ratneshpuskar/${appName.toLowerCase()}:${appVersion}"
                    
                    sh "docker build -t ${dockerImage} ."
                }
            }
        }
        stage('Docker Login and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    sh 'echo "$DOCKER_HUB_PASSWORD" | docker login --username "$DOCKER_HUB_USERNAME" --password-stdin'
                    sh "docker push ${dockerImage}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def appName = "helloservice"
                    def appVersion = "latest"
                    def appPort = "5000"
                    def serviceTemplate = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${appName}
                    spec:
                      ports:
                      - port: ${appPort}
                        protocol: TCP
                        targetPort: ${appPort}
                      selector:
                        app: ${appName}
                    """

                    def deploymentTemplate = """
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: ${appName}
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: ${appName}
                      template:
                        metadata:
                          labels:
                            app: ${appName}
                        spec:
                          containers:
                          - name: ${appName}
                            image: ${dockerImage}
                            ports:
                            - containerPort: ${appPort}
                    """

                    writeFile file: 'service.yaml', text: serviceTemplate
                    writeFile file: 'deployment.yaml', text: deploymentTemplate
                    
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 "kubectl apply -f -" < service.yaml'
                    
                    sleep(60)
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 "sudo lsof -t -i :5000 | xargs -r sudo kill -9"' || true
                    sh "ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 \"kubectl port-forward --address 0.0.0.0 service/${appName} ${appPort}:${appPort}\"" || true
                }
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