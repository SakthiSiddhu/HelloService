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

        stage('Docker Build and Push') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    def dockerImage = "ratneshpuskar/${projectName}:${dockerTag}"
                    
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh "docker build -t ${dockerImage} ."
                        sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                        sh "docker push ${dockerImage}"
                    }
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                script {
                    def deploymentYaml = '''
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: helloservice-deployment
                    spec:
                      replicas: 2
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
                            image: ratneshpuskar/helloservice:latest
                            ports:
                            - containerPort: 5000
                    '''

                    def serviceYaml = '''
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

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.201.96.193 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.201.96.193 "kubectl apply -f -" < service.yaml'
                    sleep 60
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.201.96.193 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000"'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build, Docker, and Kubernetes deployment completed successfully!'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}