pipeline {
    agent any

    tools {
        // Install the Maven version configured in Jenkins
        maven 'Maven'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout code from GitHub
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService'
            }
        }

        stage('Build Project') {
            steps {
                // Build the project with Maven, skip tests
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def projectName = "helloservice"
                    def dockerTag = "latest"
                    def dockerImageName = "ratneshpuskar/${projectName}:${dockerTag}"

                    // Build Docker image
                    sh "docker build -t ${dockerImageName} ."

                    // Securely log in to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"

                        // Push the Docker image to the repository
                        sh "docker push ${dockerImageName}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Generate deployment YAML
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
        image: ratneshpuskar/helloservice:latest
        ports:
        - containerPort: 5000
                    '''
                    writeFile file: 'deployment.yaml', text: deploymentYaml

                    // Generate service YAML
                    def serviceYaml = '''
apiVersion: v1
kind: Service
metadata:
  name: helloservice
spec:
  selector:
    app: helloservice
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
                    '''
                    writeFile file: 'service.yaml', text: serviceYaml

                    // Apply YAML files and initiate port forwarding
                    sh '''
                      ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "kubectl apply -f -" < deployment.yaml
                      ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "kubectl apply -f -" < service.yaml
                      sleep 60
                      ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true
                      ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "kubectl port-forward --address 0.0.0.0 service/helloservice 5000:5000" || true
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }

        failure {
            echo 'Deployment failed.'
        }
    }
}