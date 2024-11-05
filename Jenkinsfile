pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        KUBECONFIG = '/home/ec2-user/.kube/config'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
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
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'

                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh """
                        docker build -t ${DOCKER_USERNAME}/${projectName}:${dockerTag} .
                        echo ${DOCKER_PASSWORD} | docker login --username ${DOCKER_USERNAME} --password-stdin
                        docker push ${DOCKER_USERNAME}/${projectName}:${dockerTag}
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                writeFile file: 'deployment.yaml', text: '''\
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

                writeFile file: 'service.yaml', text: '''\
apiVersion: v1
kind: Service
metadata:
  name: helloservice-service
spec:
  selector:
    app: helloservice
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
'''

                sh '''
                ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@65.1.114.85 "kubectl apply -f deployment.yaml --validate=false"
                ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@65.1.114.85 "kubectl apply -f service.yaml --validate=false"
                '''
            }
        }

        stage('Sleep Post Deployment') {
            steps {
                sleep time: 1, unit: 'MINUTES'
            }
        }

        stage('Port Forward') {
            steps {
                sh '''
                ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@65.1.114.85 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:80" &
                sleep 120
                '''
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