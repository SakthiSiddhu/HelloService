pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService.git'
            }
        }

        stage('Build Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def projectname = "helloservice"
                    def dockertag = "latest"
                    def image = "ratneshpuskar/${projectname.toLowerCase()}:${dockertag}"

                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            docker build -t ${image} .
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            docker push ${image}
                        """
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
        image: ratneshpuskar/helloservice:latest
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

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh """
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 "kubectl apply -f -" < deployment.yaml &&
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 "kubectl apply -f -" < service.yaml &&
                        sleep 60 &&
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true &&
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@35.154.130.137 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000" || true
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