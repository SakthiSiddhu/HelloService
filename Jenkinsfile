pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub_credentials')
        PROJECT_NAME = "helloservice"
        DOCKER_TAG = "latest"
        KUBERNETES_HOST = "13.201.96.193"
        PEM_FILE_PATH = "/var/test.pem"
        SERVICE_NAME = "helloservice"
        CONTAINER_PORT = "5000"
        SERVICE_PORT = "5000"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService'
            }
        }

        stage('Build and Skip Tests') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "${env.DOCKER_CREDENTIALS_USR}/${env.PROJECT_NAME}:${env.DOCKER_TAG}".toLowerCase()
                    sh "docker build -t ${imageName} ."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]) {
                        sh """
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${imageName}
                        """
                    }
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
                      name: ${env.SERVICE_NAME}
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: ${env.SERVICE_NAME}
                      template:
                        metadata:
                          labels:
                            app: ${env.SERVICE_NAME}
                        spec:
                          containers:
                          - name: ${env.SERVICE_NAME}
                            image: ${env.DOCKER_CREDENTIALS_USR}/${env.PROJECT_NAME}:${env.DOCKER_TAG}.toLowerCase()
                            ports:
                            - containerPort: ${env.CONTAINER_PORT}
                    """

                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${env.SERVICE_NAME}
                    spec:
                      type: NodePort
                      selector:
                        app: ${env.SERVICE_NAME}
                      ports:
                      - protocol: TCP
                        port: ${env.SERVICE_PORT}
                        targetPort: ${env.CONTAINER_PORT}
                    """

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh "scp -i ${env.PEM_FILE_PATH} deployment.yaml ec2-user@${env.KUBERNETES_HOST}:~"
                    sh "scp -i ${env.PEM_FILE_PATH} service.yaml ec2-user@${env.KUBERNETES_HOST}:~"

                    sh "ssh -i ${env.PEM_FILE_PATH} -o StrictHostKeyChecking=no ec2-user@${env.KUBERNETES_HOST} 'kubectl apply -f ~/deployment.yaml'"
                    sh "ssh -i ${env.PEM_FILE_PATH} -o StrictHostKeyChecking=no ec2-user@${env.KUBERNETES_HOST} 'kubectl apply -f ~/service.yaml'"

                    sleep 60

                    sh "ssh -i ${env.PEM_FILE_PATH} -o StrictHostKeyChecking=no ec2-user@${env.KUBERNETES_HOST} 'kubectl port-forward --address 0.0.0.0 service/${env.SERVICE_NAME} ${env.CONTAINER_PORT}:${env.SERVICE_PORT}'"
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}