pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        KUBECONFIG = '/home/ec2-user/.kube/config'
    }
    stages {
        stage ('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerTag = 'latest'
                    def projectName = 'helloservice'
                    sh "docker build -t ratneshpuskar/${projectName.toLowerCase()}:${dockerTag} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    script {
                        def dockerTag = 'latest'
                        def projectName = 'helloservice'
                        sh "docker push ratneshpuskar/${projectName.toLowerCase()}:${dockerTag}"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    writeFile file: 'deployment.yaml', text: """
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
        image: ratneshpuskar/${projectName}:${dockerTag}
        ports:
        - containerPort: 5000
"""
                    writeFile file: 'service.yaml', text: """
apiVersion: v1
kind: Service
metadata:
  name: ${projectName}-service
spec:
  selector:
    app: ${projectName}
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
"""
                    sh"""
                 ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@3.6.87.166 "kubectl apply -f -" < deployment.yaml
                 ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@3.6.87.166 "kubectl apply -f -" < service.yaml
                 ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@3.6.87.166 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:80 &"
                 """
                sleep 60
                sh "exit"
                }
            }
        }
    }
}