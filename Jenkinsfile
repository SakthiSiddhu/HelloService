pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        KUBECONFIG = '/home/ec2-user/.kube/config'
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
        stage('Docker Build and Push') {
            steps {
                script {
                    def appName = "helloservice"
                    def dockerTag = "${appName}:dockerTag"
                    def dockerImage = "ratneshpuskar/${dockerTag}".toLowerCase()
                    
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                        docker build -t ${dockerImage} .
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker push ${dockerImage}
                        """
                    }
                }
            }
        }
        stage('Kubernetes Deploy') {
            steps {
                script {
                    def appName = "helloservice"
                    def dockerTag = "${appName}:dockerTag"
                    def dockerImage = "ratneshpuskar/${dockerTag}".toLowerCase()

                    sh """
                    cat <<EOF | kubectl apply -f -
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: ${appName}
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: ${appName}
                      template:
                        metadata:
                          labels:
                            app: ${appName}
                        spec:
                          containers:
                          - name: ${appName}
                            image: ${dockerImage}
                            ports:
                            - containerPort: 5000
                    ---
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${appName}-service
                    spec:
                      selector:
                        app: ${appName}
                      ports:
                      - protocol: TCP
                        port: 80
                        targetPort: 5000
                    EOF
                    """
                }
            }
        }
    }
}