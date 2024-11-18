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

        stage('Create Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh """
                       docker build -t ${imageName} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh """
                       echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin
                       docker push ratneshpuskar/helloservice:${env.BUILD_NUMBER}
                    """
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
                      selector:
                        app: helloservice
                      ports:
                        - protocol: TCP
                          port: 5000
                          targetPort: 5000
                          nodePort: 30007
                    """

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh """
                       ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.127.129.44 "kubectl apply -f -" < deployment.yaml
                       ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.127.129.44 "kubectl apply -f -" < service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}