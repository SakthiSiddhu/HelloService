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
        stage('Docker Build and Push') {
            steps {
                script {
                    def dockerTag = "${env.BUILD_ID}"
                    def imageName = "bharathkamal/helloservice:${dockerTag}"

                    sh "docker build -t ${imageName} ."
                    
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh "docker push ${imageName}"
                    }
                }
            }
        }
        stage('Kubernetes Deployment') {
            steps {
                script {
                    def dockerTag = "${env.BUILD_ID}"
                    def imageName = "bharathkamal/helloservice:${dockerTag}"

                    sh """
                    cat <<EOF > deployment.yaml
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
        image: ${imageName}
        ports:
        - containerPort: 5000
EOF
                    """

                    sh """
                    cat <<EOF > service.yaml
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
    targetPort: 5000
EOF
                    """

                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
}