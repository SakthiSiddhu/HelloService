pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        registry = "ratneshpuskar/${env.BRANCH_NAME.toLowerCase()}:${env.BUILD_NUMBER}"
        dockerHubCredentials = credentials('dockerhub_credentials')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: 'main']], 
                    userRemoteConfigs: [[url: 'https://github.com/DatlaBharath/HelloService']]
                ])
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
                    docker.withRegistry('', dockerHubCredentials) {
                        def customImage = docker.build(registry)
                        customImage.push()
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
        image: ${registry}
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
  type: LoadBalancer
      '''
                    
                    writeFile(file: 'deployment.yaml', text: deploymentYaml)
                    writeFile(file: 'service.yaml', text: serviceYaml)
                    
                    sh 'scp -i /var/test.pem -o StrictHostKeyChecking=no deployment.yaml ec2-user@15.206.72.215:/tmp/deployment.yaml'
                    sh 'scp -i /var/test.pem -o StrictHostKeyChecking=no service.yaml ec2-user@15.206.72.215:/tmp/service.yaml'
                    
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@15.206.72.215 "kubectl apply -f /tmp/deployment.yaml"'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@15.206.72.215 "kubectl apply -f /tmp/service.yaml"'
                    
                    sleep 60
                    
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@15.206.72.215 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@15.206.72.215 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000" || true'
                }
            }
        }
    }
    post {
        success {
            echo 'Job completed successfully.'
        }
        failure {
            echo 'Job failed.'
        }
    }
}