pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_credentials')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    def project = 'HelloService'.toLowerCase()
                    def dockerTag = 'latest'

                    sh "docker build -t ${DOCKERHUB_CREDENTIALS_USR}/${project}:${dockerTag} ."
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    sh "docker push ${DOCKERHUB_CREDENTIALS_USR}/${project}:${dockerTag}"
                }
            }
        }

        stage('Kubernetes Deploy') {
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
        image: ${DOCKERHUB_CREDENTIALS_USR}/helloservice:latest
        ports:
        - containerPort: 5000
                    '''
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

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < service.yaml'
                    sh 'sleep 60'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl port-forward --address 0.0.0.0 service/helloservice 5000:5000" || true'
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