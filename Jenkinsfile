pipeline {
    agent any
    tools {
        maven "Maven"
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
                    def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]) {
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USER} --password-stdin"
                    sh "docker tag ratneshpuskar/helloservice:${env.BUILD_NUMBER} ratneshpuskar/helloservice:latest"
                    sh "docker push ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh "docker push ratneshpuskar/helloservice:latest"
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
                      name: helloservice
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
                      name: helloservice
                    spec:
                      selector:
                        app: helloservice
                      ports:
                        - protocol: TCP
                          port: 5000
                          targetPort: 5000
                          nodePort: 30007
                      type: NodePort
                    """
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml
                    sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.236.171 "kubectl apply -f -" < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.236.171 "kubectl apply -f -" < service.yaml
                    """
                }
            }
        }
    }
    post {
        success {
            echo "Job completed successfully!"
        }
        failure {
            echo "Job failed!"
        }
    }
}