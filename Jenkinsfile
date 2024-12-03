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
                    def dockerImageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh """
                    docker build -t ${dockerImageName} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh """
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    docker push ratneshpuskar/helloservice:${env.BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sshagent(credentials: ['jenkins_ssh_key']) {
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
                              - name: helloservice
                                image: ratneshpuskar/helloservice:${env.BUILD_NUMBER}
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
                            nodePort: 30007
                          type: NodePort
                        """

                        sh """
                        echo "${deploymentYaml}" > deployment.yaml
                        echo "${serviceYaml}" > service.yaml
                        ssh -o StrictHostKeyChecking=no -i /var/test.pem ubuntu@<k8s_instance_ip> "kubectl apply -f -" < deployment.yaml
                        ssh -o StrictHostKeyChecking=no -i /var/test.pem ubuntu@<k8s_instance_ip> "kubectl apply -f -" < service.yaml
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline finished successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}