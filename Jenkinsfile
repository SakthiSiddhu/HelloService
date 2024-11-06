pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        DOCKER_IMAGE = "ratneshpuskar/helloservice:dockertag"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/DatlaBharath/HelloService']]])
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def projectName = 'helloservice'
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }
        stage('Push Docker Image') {
            environment {
                DOCKER_CREDENTIALS_ID = 'dockerhub_credentials'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                    sh 'docker push ${DOCKER_IMAGE}'
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
                            image: ${DOCKER_IMAGE}
                            ports:
                            - containerPort: 5000
                    """
                    writeFile file: 'deployment.yaml', text: deploymentYaml

                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: helloservice
                    spec:
                      selector:
                        app: helloservice
                      ports:
                        - protocol: TCP
                          port: 80
                          targetPort: 5000
                    """
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh 'scp -o StrictHostKeyChecking=no -i /var/test.pem deployment.yaml ec2-user@52.66.182.58:/home/ec2-user/'
                    sh 'scp -o StrictHostKeyChecking=no -i /var/test.pem service.yaml ec2-user@52.66.182.58:/home/ec2-user/'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.182.58 "kubectl apply -f /home/ec2-user/deployment.yaml &"'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.182.58 "kubectl apply -f /home/ec2-user/service.yaml &"'
                     sleep 60
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.182.58 "kubectl port-forward --address 0.0.0.0 service/helloservice 5000:80"'
                   
                }
            }
        }
    }
    post {
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}