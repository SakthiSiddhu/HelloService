pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('Clone repository') {
            steps {
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
            }
        }
        stage('Build the project') {
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
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin'
                        def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYAML = '''
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: hello-deployment
                    spec:
                      replicas: 2
                      selector:
                        matchLabels:
                          app: hello-app
                      template:
                        metadata:
                          labels:
                            app: hello-app
                        spec:
                          containers:
                          - name: hello-container
                            image: ratneshpuskar/helloservice:${env.BUILD_NUMBER}
                            ports:
                            - containerPort: 5000
                    '''
                    def serviceYAML = '''
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: hello-service
                    spec:
                      type: NodePort
                      selector:
                        app: hello-app
                      ports:
                      - protocol: TCP
                        port: 5000
                        targetPort: 5000
                        nodePort: 30007
                    '''
                    writeFile file: 'deployment.yaml', text: deploymentYAML
                    writeFile file: 'service.yaml', text: serviceYAML
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.126.245.129 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.126.245.129 "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }
    post {
        success {
            echo 'Build and deployment completed successfully.'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}