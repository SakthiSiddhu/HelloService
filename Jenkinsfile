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
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def mavenProject = 'helloservice'
                    sh "docker build -t ratneshpuskar/${mavenProject.toLowerCase()}:dockertag ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    def mavenProject = 'helloservice'
                    sh "docker push ratneshpuskar/${mavenProject.toLowerCase()}:dockertag"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Create deployment.yaml and service.yaml
                    writeFile file: 'deployment.yaml', text: '''
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
                    '''
                    writeFile file: 'service.yaml', text: '''
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
                    '''

                    // Apply deployment.yaml and service.yaml on the Kubernetes cluster
                    sh """
                      ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 "kubectl apply -f -" < deployment.yaml
                      ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 "kubectl apply -f -" < service.yaml
                      sleep 60
                      ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true
                      ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000" || true
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}