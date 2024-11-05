pipeline {
    agent any
    tools { maven 'Maven' }
    environment {
        KUBECONFIG = '/home/ec2-user/.kube/config'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = "latest"
                    def imageName = "ratneshpuskar/${projectName.toLowerCase()}:${dockerTag}"
                    
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhubpwd', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        def projectName = 'helloservice'
                        def dockerTag = "latest"
                        def imageName = "ratneshpuskar/${projectName.toLowerCase()}:${dockerTag}"
                        
                        sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def projectName = 'helloservice'
                    def dockerTag = "latest"
                    def imageName = "ratneshpuskar/${projectName.toLowerCase()}:${dockerTag}"
                    
                    sh """
                    cat <<EOF > deployment.yaml
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: ${projectName}-deployment
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: ${projectName}
                      template:
                        metadata:
                          labels:
                            app: ${projectName}
                        spec:
                          containers:
                          - name: ${projectName}
                            image: ${imageName}
                            ports:
                            - containerPort: 5000
                    EOF
                    """
                    
                    sh """
                    cat <<EOF > service.yaml
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${projectName}-service
                    spec:
                      selector:
                        app: ${projectName}
                      ports:
                      - protocol: TCP
                        port: 5000
                        targetPort: 5000
                    EOF
                    """
                    
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"
                }
            }
        }
    }
}