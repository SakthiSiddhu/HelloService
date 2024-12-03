pipeline {
    agent any

    tools {
        // Example: Define your build tools, e.g. Maven or other
        // maven 'M3'
        // Remove this and add the correct tool
    }

    stages {
        stage('Checkout') {
            steps {
                // Replace with your specific SCM checkout
                checkout([$class: 'GitSCM', branches: [[name: '*/your-branch']], userRemoteConfigs: [[url: 'your-repo-url']]])
            }
        }

        stage('Build') {
            steps {
                // Assuming you are using Maven. Skip the tests during build
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    def app = "ratneshpuskar/your-repo-name".toLowerCase()
                    def commitId = env.BUILD_NUMBER

                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUsername')]) {
                        sh "docker build -t ${app}:${commitId} ."
                        sh "docker login -u \"${dockerUsername}\" -p \"${dockerPassword}\""
                        sh "docker push ${app}:${commitId}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def app = "ratneshpuskar/your-repo-name".toLowerCase()
                    def commitId = env.BUILD_NUMBER

                    writeFile file: 'deployment.yaml', text: """
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: your-app
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: your-app
                      template:
                        metadata:
                          labels:
                            app: your-app
                        spec:
                          containers:
                          - name: your-app
                            image: ${app}:${commitId}
                            ports:
                            - containerPort: 80
                    """

                    writeFile file: 'service.yaml', text: """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: your-app-service
                    spec:
                      selector:
                        app: your-app
                      ports:
                        - protocol: TCP
                          port: 80
                          targetPort: 80
                          nodePort: 30007
                      type: NodePort
                    """
                    sh """
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@your-k8s-host "kubectl apply -f -" < deployment.yaml
                    ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@your-k8s-host "kubectl apply -f -" < service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}