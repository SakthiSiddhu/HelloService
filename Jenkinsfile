pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        KUBECONFIG = '/home/ec2-user/.kube/config'
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
                    def imageName = "ratneshpuskar/helloservice:dockertag"
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'docker push ratneshpuskar/helloservice:dockertag'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    writeFile file: 'deployment.yaml', text: """
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

                    writeFile file: 'service.yaml', text: """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: helloservice-service
                    spec:
                      ports:
                      - port: 5000
                        targetPort: 5000
                      selector:
                        app: helloservice
                    """

                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@65.1.114.85 kubectl apply --validate=false -f - < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@65.1.114.85 kubectl apply --validate=false -f - < service.yaml'
                }
            }
        }
        stage('Sleep') {
            steps {
                script {
                    sleep 60
                }
            }
        }
        stage('Port Forward') {
            steps {
                script {
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@65.1.114.85 kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000'
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment succeeded'
        }
        failure {
            echo 'Deployment failed'
        }
    }
}