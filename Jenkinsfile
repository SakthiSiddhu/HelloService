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
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def projectName = 'hello-service'.toLowerCase()
                    def dockerTag = "${env.BUILD_NUMBER}"
                    sh """
                        docker build -t ${DOCKERHUB_CREDENTIALS_USR}/${projectName}:${dockerTag} .
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKERHUB_CREDENTIALS_USR}/${projectName}:${dockerTag}
                    """
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
                      name: hello-service
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: hello-service
                      template:
                        metadata:
                          labels:
                            app: hello-service
                        spec:
                          containers:
                          - name: hello-service
                            image: ${DOCKERHUB_CREDENTIALS_USR}/hello-service:${dockerTag}
                            ports:
                            - containerPort: 5000
                    '''
                    def serviceYaml = '''
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
                      type: LoadBalancer
                    '''
                    writeFile(file: 'deployment.yaml', text: deploymentYaml)
                    writeFile(file: 'service.yaml', text: serviceYaml)

                    sh """
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.126.196.29 "kubectl apply -f -" < deployment.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.126.196.29 "kubectl apply -f -" < service.yaml
                        sleep 60
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.126.196.29 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.126.196.29 "kubectl port-forward --address 0.0.0.0 service/hello-service 5000:5000" || true
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}