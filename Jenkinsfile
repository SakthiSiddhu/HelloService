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
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def projectName = 'helloservice'.toLowerCase()
                    def dockerTag = 'dockertag'
                    def dockerImage = "${env.DOCKERHUB_CREDENTIALS_USR}/${projectName}:${dockerTag}"
                    sh "docker build -t ${dockerImage} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def projectName = 'helloservice'.toLowerCase()
                    def dockerTag = 'dockertag'
                    def dockerImage = "${env.DOCKERHUB_CREDENTIALS_USR}/${projectName}:${dockerTag}"
                    sh "echo ${env.DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${env.DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    sh "docker push ${dockerImage}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deployment = '''
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
                            image: ${env.DOCKERHUB_CREDENTIALS_USR}/helloservice:dockertag
                            ports:
                            - containerPort: 5000
                    '''
                    def service = '''
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: helloservice
                    spec:
                      selector:
                        app: helloservice
                      ports:
                        - protocol: TCP
                          port: 80
                          targetPort: 5000
                    '''

                    writeFile file: 'deployment.yaml', text: deployment
                    writeFile file: 'service.yaml', text: service

                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < service.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "sudo lsof -t -i :5000 | xargs -r sudo kill -9"  || true'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl port-forward --address 0.0.0.0 service/helloservice 5000:80"'
                }
            }
        }
    }

    post {
        success {
            echo 'Build and Deployment completed successfully!'
        }
        failure {
            echo 'Build or Deployment failed!'
        }
    }
}