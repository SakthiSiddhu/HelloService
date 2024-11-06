pipeline {
    agent any

    tools {
        maven "Maven"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], 
                    userRemoteConfigs: [[url: 'https://github.com/DatlaBharath/HelloService']]
                ])
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
                    def projectName = "HelloService".toLowerCase()
                    def dockerTag = "latest"
                    def imageName = "ratneshpuskar/${projectName}:${dockerTag}"
                    sh """
                    docker build -t ${imageName} .
                    docker tag ${imageName} ratneshpuskar/${projectName}:${dockerTag}
                    """
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                    echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                    docker push ratneshpuskar/helloservice:latest
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
                          - name: helloservice-container
                            image: ratneshpuskar/helloservice:latest
                            ports:
                            - containerPort: 5000
                    """

                    def serviceYaml = """
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
                    """

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < service.yaml
                    sleep 60
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000" || true
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}