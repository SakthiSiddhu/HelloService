pipeline {
    agent {
        label 'linux'
    }
    tools {
        maven 'Maven'
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
        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
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
  name: helloservice
spec:
  replicas: 3
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
  name: helloservice
spec:
  type: NodePort
  selector:
    app: helloservice
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
    nodePort: 30007
"""
                    sh """ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.126.24.151 'echo "${deploymentYaml}" | kubectl apply -f -'"""
                    sh """ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.126.24.151 'echo "${serviceYaml}" | kubectl apply -f -'"""
                }
            }
        }
    }
    post {
        success {
            echo 'Job completed successfully!'
        }
        failure {
            echo 'Job failed.'
        }
    }
}