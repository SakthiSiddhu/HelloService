pipeline 
    agent any
    
    tools {
        maven 'Maven'
    }
    
    stages {
        stage('Checkout Code') {
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
                    def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        
        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'dockerhubPassword', usernameVariable: 'dockerhubUsername')]) {
                    script {
                        def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                        sh "echo ${dockerhubPassword} | docker login -u ${dockerhubUsername} --password-stdin"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
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
  type: NodePort
  selector:
    app: helloservice
  ports:
    - port: 5000
      nodePort: 30007
"""
                    sh """
                        echo "${deploymentYaml}" > deployment.yaml
                        echo "${serviceYaml}" > service.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.201.34.181 "kubectl apply -f -" < deployment.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.201.34.181 "kubectl apply -f -" < service.yaml
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment was successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
