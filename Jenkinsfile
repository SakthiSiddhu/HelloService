pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: 'main']], userRemoteConfigs: [[url: 'https://github.com/DatlaBharath/HelloService']]])
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
                    def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                    sh """
                    docker build -t ${imageName} .
                    """
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                    script {
                        def imageName = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"
                        sh """
                        echo ${DOCKERHUB_PASSWORD} | docker login -u ${DOCKERHUB_USERNAME} --password-stdin
                        docker push ${imageName}
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
                      name: helloservice
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
                            image: ratneshpuskar/helloservice:${env.BUILD_NUMBER}
                            ports:
                            - containerPort: 5000
                    """

                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: helloservice
                    spec:
                      type: NodePort
                      ports:
                        - port: 5000
                          targetPort: 5000
                          nodePort: 30007
                      selector:
                        app: helloservice
                    """

                    sh """
                    echo '${deploymentYaml}' | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.126.245.129 "kubectl apply -f -"
                    echo '${serviceYaml}' | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.126.245.129 "kubectl apply -f -"
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Build and Deployment Succeeded.'
        }
        failure {
            echo 'Build or Deployment Failed.'
        }
    }
}