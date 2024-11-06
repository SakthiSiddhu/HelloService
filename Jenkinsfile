pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/DatlaBharath/HelloService']]])
            }
        }
        stage('Build Maven Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/helloservice"
                    sh 'docker build -t ${imageName}:dockertag .'
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                        sh 'docker push ${imageName}:dockertag'
                    }
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
                    sh 'scp -i /var/test.pem -o StrictHostKeyChecking=no deployment.yaml ec2-user@52.66.182.58:/home/ec2-user/'
                    sh 'scp -i /var/test.pem -o StrictHostKeyChecking=no service.yaml ec2-user@52.66.182.58:/home/ec2-user/'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.182.58 "kubectl apply -f /home/ec2-user/deployment.yaml"'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.182.58 "kubectl apply -f /home/ec2-user/service.yaml"'
                    sleep 60
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.182.58 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000"'
                    
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}