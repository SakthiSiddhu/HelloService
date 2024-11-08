pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
            }
        }
        stage('Build with Maven') {
            steps {
                script {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('ratneshpuskar/helloservice:dockertag')
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        sh 'echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin'
                        sh 'docker push ratneshpuskar/helloservice:dockertag'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    writeFile file: 'deployment.yaml', text: """
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
        image: ratneshpuskar/helloservice:dockertag
        ports:
        - containerPort: 5000
"""
                    writeFile file: 'service.yaml', text: """
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
  type: LoadBalancer
"""
                    sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.233.18.108 "kubectl apply -f -" < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.233.18.108 "kubectl apply -f -" < service.yaml
                    sleep 60
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.233.18.108 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.233.18.108 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000" || true
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}