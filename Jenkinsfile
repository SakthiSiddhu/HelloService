pipeline {
    agent any
    
    tools {
        maven 'Maven_3.9.9'
    }
    
    environment {
        KUBECONFIG = '/home/ec2-user/.kube/config'
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
            }
        }
        
        stage('Build Maven Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    def imageName = "bharathkamal/${projectName.toLowerCase()}:${dockerTag}"
                    
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                    script {
                        def projectName = 'helloservice'
                        def dockerTag = 'latest'
                        def imageName = "bharathkamal/${projectName.toLowerCase()}:${dockerTag}"
                        
                        sh "echo ${DOCKERHUB_PASSWORD} | docker login -u ${DOCKERHUB_USERNAME} --password-stdin"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    def imageName = "bharathkamal/${projectName.toLowerCase()}:${dockerTag}"
                    def deploymentYaml = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${projectName}
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
          image: ${imageName}
          ports:
            - containerPort: 5000
                    """
                    def serviceYaml = """
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
                    """
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml
                    
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
}