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
        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = 'ratneshpuskar/helloservice:${env.BUILD_ID.toLowerCase()}'
                    sh "docker build -t ${imageName} ."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                        sh """
                        echo ${DOCKERHUB_PASSWORD} | docker login -u ${DOCKERHUB_USERNAME} --password-stdin
                        docker push ${imageName}
                        docker logout
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
                                  image: ratneshpuskar/helloservice:${env.BUILD_ID.toLowerCase()}
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
                    writeFile(file: 'deployment.yaml', text: deploymentYaml)
                    writeFile(file: 'service.yaml', text: serviceYaml)
                    sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.182.58 "kubectl apply -f -" < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.182.58 "kubectl apply -f -" < service.yaml
                    sleep 60
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.182.58 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000"
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Job succeeded!'
        }
        failure {
            echo 'Job failed!'
        }
    }
}