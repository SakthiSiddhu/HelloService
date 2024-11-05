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
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = "ratneshpuskar/helloservice:dockertag"
                    sh "docker build -t ${dockerImage} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                          docker push ratneshpuskar/helloservice:dockertag'''
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYaml = '''
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
                    '''
                    def serviceYaml = '''
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
                    '''
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.124.162 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.124.162 "kubectl apply -f -" < service.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.124.162 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000"'
                }
            }
        }
    }
    post {
        success {
            echo 'Build and deployment succeeded.'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}