pipeline {
    agent any

    tools {
        maven "Maven"
        docker "docker"
    }

    environment {
        KUBECONFIG = "/home/ec2-user/.kube/config"
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
                    def projectName = 'helloservice'
                    def dockerTag = "${env.BUILD_NUMBER}"
                    sh "docker build -t ratneshpuskar/${projectName}:${dockerTag} ."
                }
            }
        }
        stage('Docker Login and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ratneshpuskar/helloservice:${BUILD_NUMBER}
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def imageTag = "ratneshpuskar/helloservice:${BUILD_NUMBER}"
                    sh """
                        echo "
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
                                image: ${imageTag}
                                ports:
                                - containerPort: 5000
                        ---
                        apiVersion: v1
                        kind: Service
                        metadata:
                          name: helloservice-service
                        spec:
                          selector:
                            app: helloservice
                          ports:
                            - protocol: TCP
                              port: 80
                              targetPort: 5000
                        " > k8s_deployment.yaml
                        kubectl apply -f k8s_deployment.yaml
                    """
                }
            }
        }
    }
}