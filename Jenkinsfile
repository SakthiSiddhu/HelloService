pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub_credentials'
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
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
                    def dockerImage = "ratneshpuskar/helloservice:${BUILD_NUMBER}"
                    sh "docker build -t ${dockerImage} ."
                }
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    def dockerImage = "ratneshpuskar/helloservice:${BUILD_NUMBER}"
                    sh "docker push ${dockerImage}"
                }
            }
        }
        stage('Deployment') {
            steps {
                script {
                    sh '''
                    cat > deployment.yaml <<EOF
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
                            image: ratneshpuskar/helloservice:${BUILD_NUMBER}
                            ports:
                            - containerPort: 5000
                    EOF

                    cat > service.yaml <<EOF
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: helloservice-service
                    spec:
                      type: NodePort
                      selector:
                        app: helloservice
                      ports:
                        - protocol: TCP
                          port: 5000
                          targetPort: 5000
                    EOF

                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.126.196.29 "kubectl apply -f -" < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.126.196.29 "kubectl apply -f -" < service.yaml
                    sleep 60
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.126.196.29 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.126.196.29 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000" || true
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Job finished successfully.'
        }
        failure {
            echo 'Job failed.'
        }
    }
}