pipeline {
    agent any

    tools {
        maven 'Maven_3.9.9'
    }

    environment {
        KUBECONFIG = '/home/ec2-user/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                git(url: 'https://github.com/DatlaBharath/HelloService', branch: 'main')
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
                    def projectName = env.GIT_URL.tokenize('/')[3].split("\\.")[0].toLowerCase()
                    def dockerTag = "${env.BUILD_NUMBER}"
                    def imageName = "bharathkamal/${projectName}:${dockerTag}"

                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    script {
                        def projectName = env.GIT_URL.tokenize('/')[3].split("\\.")[0].toLowerCase()
                        def dockerTag = "${env.BUILD_NUMBER}"
                        def imageName = "bharathkamal/${projectName}:${dockerTag}"

                        sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def projectName = env.GIT_URL.tokenize('/')[3].split("\\.")[0].toLowerCase()
                    def dockerTag = "${env.BUILD_NUMBER}"
                    def imageName = "bharathkamal/${projectName}:${dockerTag}"

                    def deploymentYaml = """
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: ${projectName}-deployment
                    spec:
                      replicas: 1
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

                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${projectName}-service
                    spec:
                      selector:
                        app: ${projectName}
                      ports:
                        - protocol: TCP
                          port: 5000
                          targetPort: 5000
                    """

                    writeFile file: 'k8s-deployment.yaml', text: deploymentYaml
                    writeFile file: 'k8s-service.yaml', text: serviceYaml

                    sh 'kubectl apply -f k8s-deployment.yaml'
                    sh 'kubectl apply -f k8s-service.yaml'
                }
            }
        }
    }
}