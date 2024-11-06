```
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
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    def imageName = 'ratneshpuskar/hello-service'
                    def dockerTag = "${imageName}:${env.BUILD_NUMBER}"
                    
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "docker build -t ${dockerTag} ."
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        sh "docker push ${dockerTag}"
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deployment = """
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: hello-service-deployment
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
                            image: ratneshpuskar/hello-service:${env.BUILD_NUMBER}
                            ports:
                            - containerPort: 5000
                    """
                    
                    def service = """
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
                    
                    writeFile file: 'deployment.yaml', text: deployment.trim()
                    writeFile file: 'service.yaml', text: service.trim()

                    sh '''
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < deployment.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < service.yaml
                        sleep 60
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "sudo kill -9 $(sudo lsof -t -i :5000)"
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl port-forward --address 0.0.0.0 service/hello-service 5000:5000"
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment was successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
```