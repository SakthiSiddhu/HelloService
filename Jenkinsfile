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
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Build and Tag Docker Image') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    def dockerImage = "bharathkamal/${projectName.toLowerCase()}:${dockerTag}"
                    
                    sh """
                        docker build -t ${dockerImage} .
                    """
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    def dockerImage = "bharathkamal/${projectName.toLowerCase()}:${dockerTag}"
                    
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                            docker push ${dockerImage}
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    def dockerImage = "bharathkamal/${projectName.toLowerCase()}:${dockerTag}"
                    def deploymentName = '${projectName.toLowerCase()}-deployment'
                    
                    sh """
                        cat <<EOF > deployment.yaml
                        apiVersion: apps/v1
                        kind: Deployment
                        metadata:
                          name: ${deploymentName}
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
                                image: ${dockerImage}
                                ports:
                                - containerPort: 5000
                        EOF
                        
                        cat <<EOF > service.yaml
                        apiVersion: v1
                        kind: Service
                        metadata:
                          name: ${projectName}-service
                        spec:
                          type: LoadBalancer
                          ports:
                          - port: 5000
                            targetPort: 5000
                          selector:
                            app: ${projectName}
                        EOF
                        
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                    """
                }
            }
        }
    }
}