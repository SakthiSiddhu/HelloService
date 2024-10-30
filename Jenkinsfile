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
                git branch: 'main',
                    url: 'https://github.com/DatlaBharath/HelloService.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def projectname = 'helloservice'
                    def dockerTag = "latest"
                    sh "docker build -t ratneshpuskar/${projectname}:${dockerTag} ."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push ratneshpuskar/${projectname}:${dockerTag}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def projectname = 'helloservice'
                    def dockerTag = "latest"
                    sh """
                        cat <<EOF > deployment.yml
                        apiVersion: apps/v1
                        kind: Deployment
                        metadata:
                          name: ${projectname}
                        spec:
                          replicas: 1
                          selector:
                            matchLabels:
                              app: ${projectname}
                          template:
                            metadata:
                              labels:
                                app: ${projectname}
                            spec:
                              containers:
                              - name: ${projectname}
                                image: ratneshpuskar/${projectname}:${dockerTag}
                                ports:
                                - containerPort: 5000
                        EOF
                    """
                    sh """
                        cat <<EOF > service.yml
                        kind: Service
                        apiVersion: v1
                        metadata:
                          name: ${projectname}-service
                        spec:
                          selector:
                            app: ${projectname}
                          ports:
                            - protocol: TCP
                              port: 5000
                              targetPort: 5000
                        EOF
                    """
                    sh 'kubectl apply -f deployment.yml'
                    sh 'kubectl apply -f service.yml'
                }
            }
        }
    }
}