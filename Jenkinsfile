pipeline {
    agent any
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
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh 'echo ${DOCKERHUB_PASSWORD} | docker login -u ${DOCKERHUB_USER} --password-stdin'
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
  name: helloservice
spec:
  type: NodePort
  ports:
    - port: 5000
      nodePort: 30007
  selector:
    app: helloservice
                    """

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@<KubernetesServer> "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@<KubernetesServer> "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }
    post {
        success {
            echo 'Job completed successfully!'
        }
        failure {
            echo 'Job failed!'
        }
    }
}