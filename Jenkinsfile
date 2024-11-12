pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService.git'
            }
        }
        stage('Build Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def repoName = 'helloservice'
                    def imageName = "ratneshpuskar/${repoName.toLowerCase()}:${env.BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                        sh '''
                            docker build -t ${imageName} .
                            echo "${DOCKERHUB_PASS}" | docker login -u "${DOCKERHUB_USER}" --password-stdin
                            docker push ${imageName}
                        '''
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def repoName = 'helloservice'
                    def imageName = "ratneshpuskar/${repoName.toLowerCase()}:${env.BUILD_NUMBER}"
                    sh """
                        cat <<EOF > deployment.yaml
                        apiVersion: apps/v1
                        kind: Deployment
                        metadata:
                          name: ${repoName}
                        spec:
                          replicas: 1
                          selector:
                            matchLabels:
                              app: ${repoName}
                          template:
                            metadata:
                              labels:
                                app: ${repoName}
                            spec:
                              containers:
                              - name: ${repoName}
                                image: ${imageName}
                                ports:
                                - containerPort: 5000
                        EOF

                        cat <<EOF > service.yaml
                        apiVersion: v1
                        kind: Service
                        metadata:
                          name: ${repoName}
                        spec:
                          type: NodePort
                          selector:
                            app: ${repoName}
                          ports:
                          - protocol: TCP
                            port: 5000
                            nodePort: 30007
                        EOF
                    """
                    sh '''
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.200.252.168 "kubectl apply -f -" < deployment.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.200.252.168 "kubectl apply -f -" < service.yaml
                    '''
                }
            }
        }
    }
    post {
        success {
            echo "Job succeeded"
        }
        failure {
            echo "Job failed"
        }
    }
}