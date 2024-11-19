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
        stage('Build') {
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
                    script {
                        def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                        sh "echo ${DOCKER_HUB_PASSWORD}"
                        sh "docker login -u ${DOCKER_HUB_USERNAME} -p ${DOCKER_HUB_PASSWORD}"
                        sh "docker push ${imageName}"
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
                    def serviceYaml = """
                        apiVersion: v1
                        kind: Service
                        metadata:
                          name: helloservice-service
                        spec:
                          type: NodePort
                          ports:
                            - port: 5000
                              nodePort: 30007
                          selector:
                            app: helloservice
                    """
                    writeFile(file: 'deployment.yaml', text: deploymentYaml)
                    writeFile(file: 'service.yaml', text: serviceYaml)
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.235.113.223 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.235.113.223 "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }
    post {
        success {
            echo 'Job completed successfully!'
        }
        failure {
            echo 'Job failed.'
        }
    }
}