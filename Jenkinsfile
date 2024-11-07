pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImageName = "ratneshpuskar/${env.JOB_NAME.toLowerCase()}:dockertag"
                    sh "docker build -t ${dockerImageName} ."
                    
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                        sh "docker push ${dockerImageName}"
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
  name: hello-service-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-service
  template:
    metadata:
      labels:
        app: hello-service
    spec:
      containers:
      - name: hello-service-container
        image: ratneshpuskar/${env.JOB_NAME.toLowerCase()}:dockertag
        ports:
        - containerPort: 5000
"""
                    def serviceYaml = """
apiVersion: v1
kind: Service
metadata:
  name: hello-service
spec:
  selector:
    app: hello-service
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
"""
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh """
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "kubectl apply -f -" < deployment.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "kubectl apply -f -" < service.yaml
                        sleep 60
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "kubectl port-forward --address 0.0.0.0 service/hello-service 5000:5000" || true
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment completed successfully'
        }
        failure {
            echo 'Build or deployment failed'
        }
    }
}