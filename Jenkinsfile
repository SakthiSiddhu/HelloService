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
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh """
                        docker build -t ${dockerImage} .
                    """
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    script {
                        def dockerImage = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                        sh """
                            echo "${DOCKER_HUB_PASSWORD}" | docker login -u "${DOCKER_HUB_USERNAME}" --password-stdin
                            docker push ${dockerImage}
                        """
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def dockerImage = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    def deploymentYAML = """
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
                                image: ${dockerImage}
                                ports:
                                - containerPort: 5000
                    """
                    def serviceYAML = """
                        apiVersion: v1
                        kind: Service
                        metadata:
                          name: helloservice
                        spec:
                          type: NodePort
                          selector:
                            app: helloservice
                          ports:
                            - protocol: TCP
                              port: 5000
                              nodePort: 30007
                    """
                    sh """
                        echo '${deploymentYAML}' | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@15.206.158.106 "kubectl apply -f -"
                        echo '${serviceYAML}' | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@15.206.158.106 "kubectl apply -f -"
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}