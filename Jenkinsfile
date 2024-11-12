pipeline {
    agent any
    
    tools {
        maven 'Maven'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: 'main']], userRemoteConfigs: [[url: 'https://github.com/DatlaBharath/HelloService']]])
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
                    def repoName = "helloservice".toLowerCase()
                    def tag = "ratneshpuskar/${repoName}:${env.BUILD_NUMBER}"
                    sh """
                    docker build -t ${tag} .
                    """
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASS', usernameVariable: 'DOCKER_HUB_USER')]) {
                    sh 'echo $DOCKER_HUB_PASS | docker login -u $DOCKER_HUB_USER --password-stdin'
                    sh """
                    def repoName = "helloservice".toLowerCase()
                    def tag = "ratneshpuskar/${repoName}:${env.BUILD_NUMBER}"
                    docker push ${tag}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def repoName = "helloservice".toLowerCase()
                    def tag = "ratneshpuskar/${repoName}:${env.BUILD_NUMBER}"
                    def deploymentYaml = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${repoName}
  labels:
    app: ${repoName}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${repoName}
  template:
    metadata:
      labels:
        app: ${repoName}
    spec:
      containers:
      - name: ${repoName}
        image: ${tag}
        ports:
        - containerPort: 5000
                    """
                    
                    def serviceYaml = """
apiVersion: v1
kind: Service
metadata:
  name: ${repoName}
spec:
  selector:
    app: ${repoName}
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
    nodePort: 30007
  type: NodePort
                    """
                    
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml
                    
                    sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@35.154.5.231 'kubectl apply -f -' < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@35.154.5.231 'kubectl apply -f -' < service.yaml
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build and deploy succeeded.'
        }
        
        failure {
            echo 'Build or deploy failed.'
        }
    }
}