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
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Docker Build and Push') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = 'latest'
                    def image = "ratneshpuskar/${projectName}:${dockerTag}"
                    
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        docker build -t ${image} .
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${image}
                        """
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
                    """

                    def serviceYaml = """
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
                    """

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "kubectl apply -f -" < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "kubectl apply -f -" < service.yaml
                    sleep 60
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.232.139.240 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000" 
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment Success'
        }
        failure {
            echo 'Deployment Failed'
        }
    }
}