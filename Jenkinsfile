pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def app = docker.build("ratneshpuskar/helloservice:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push ratneshpuskar/helloservice:${env.BUILD_NUMBER}'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deployment = """
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
                                - containerPort: 80
                    """
                    def service = """
                        apiVersion: v1
                        kind: Service
                        metadata:
                          name: helloservice-service
                        spec:
                          selector:
                            app: helloservice
                          ports:
                            - protocol: TCP
                              port: 80
                              targetPort: 80
                              nodePort: 30007
                          type: NodePort
                    """

                    sh 'echo "${deployment}' | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@65.2.9.62 "kubectl apply -f -" """
                    sh 'echo "${service}' | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@65.2.9.62 "kubectl apply -f -" """
                }
            }
        }
    }
    post {
        success {
            echo 'Build and deployment completed successfully!'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}