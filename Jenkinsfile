pipeline {
    agent any

    tools {
        maven 'Maven 3.6.3' // Change to the appropriate Maven version
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/your_repo.git'
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
                    def imageName = "ratneshpuskar/your_repo_name: ${env.BUILD_NUMBER}".toLowerCase()
                    sh """
                        docker build -t ${imageName} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin'
                        def imageName = "ratneshpuskar/your_repo_name:${env.BUILD_NUMBER}".toLowerCase()
                        sh 'docker push ${imageName}'
                    }
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
                      name: your-app
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: your-app
                      template:
                        metadata:
                          labels:
                            app: your-app
                        spec:
                          containers:
                          - name: your-container
                            image: ratneshpuskar/your_repo_name:${env.BUILD_NUMBER}
                            ports:
                            - containerPort: 8080
                    """

                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: your-service
                    spec:
                      type: NodePort
                      ports:
                        - port: 8080
                          targetPort: 8080
                          nodePort: 30007
                      selector:
                        app: your-app
                    """

                    sh """
                        echo '${deploymentYaml}' > deployment.yaml
                        echo '${serviceYaml}' > service.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@<INSTANCE_2_IP> "kubectl apply -f -" < deployment.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@<INSTANCE_2_IP> "kubectl apply -f -" < service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}