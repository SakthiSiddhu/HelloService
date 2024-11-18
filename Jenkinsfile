pipeline {
    agent any

    tools {
        nodejs "NodeJS"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/DatlaBharath/docker-react', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build -- --skip-tests'
            }
        }

        stage('Create Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    script {
                        def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                        sh 'docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
                        sh "docker push ${imageName}"
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
                      name: docker-react-deployment
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: docker-react
                      template:
                        metadata:
                          labels:
                            app: docker-react
                        spec:
                          containers:
                          - name: docker-react
                            image: ratneshpuskar/docker-react:${env.BUILD_NUMBER}
                            ports:
                            - containerPort: 80
                    """
                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: docker-react-service
                    spec:
                      type: NodePort
                      selector:
                        app: docker-react
                      ports:
                      - protocol: TCP
                        port: 80
                        targetPort: 80
                        nodePort: 30007
                    """
                    
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    // Apply deployment.yaml file
                    sh "ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.127.129.44 \"kubectl apply -f -\" < deployment.yaml"

                    // Apply service.yaml file
                    sh "ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.127.129.44 \"kubectl apply -f -\" < service.yaml"
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