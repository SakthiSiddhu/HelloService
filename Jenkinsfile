pipeline {
    agent any
    tools {
        maven "Maven"
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
                    def dockerImageTag = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh 'docker build -t ${dockerImageTag} .'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        sh 'echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin'
                        sh 'docker push ratneshpuskar/helloservice:${env.BUILD_NUMBER}'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYaml = '''
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
                    '''
                    writeFile file: 'deployment.yaml', text: deploymentYaml

                    def serviceYaml = '''
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: helloservice-service
                    spec:
                      type: NodePort
                      selector:
                        app: helloservice
                      ports:
                        - port: 5000
                          nodePort: 30007
                    '''
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@65.2.190.63 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@65.2.190.63 "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }
    post {
        success {
            echo 'Build and deployment successful'
        }
        failure {
            echo 'Build or deployment failed'
        }
    }
}