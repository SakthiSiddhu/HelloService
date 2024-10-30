pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        KUBECONFIG = "/home/ec2-user/.kube/config"
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
                    def projectName = 'helloservice' // derived from repo URL
                    def dockerTag = 'latest'
                    def imageName = "ratneshpuskar/${projectName}:${dockerTag}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        def projectName = 'helloservice'
                        def dockerTag = 'latest'
                        def imageName = "ratneshpuskar/${projectName}:${dockerTag}"
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    def imageName = "ratneshpuskar/${projectName}:${dockerTag}"
                    def deploymentYaml = """
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: ${projectName}-deployment
                    spec:
                      replicas: 2
                      selector:
                        matchLabels:
                          app: ${projectName}
                      template:
                        metadata:
                          labels:
                            app: ${projectName}
                        spec:
                          containers:
                          - name: ${projectName}
                            image: ${imageName}
                            ports:
                            - containerPort: 5000
                    """
                    sh "echo '${deploymentYaml}' > deployment.yaml"

                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${projectName}-service
                    spec:
                      type: NodePort
                      selector:
                        app: ${projectName}
                      ports:
                        - protocol: TCP
                          port: 5000
                          targetPort: 5000
                    """
                    sh "echo '${serviceYaml}' > service.yaml"
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"
                }
            }
        }
    }
}