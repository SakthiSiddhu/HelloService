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
        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/${env.JOB_NAME.toLowerCase()}:${env.BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')])
                    {
                        sh "docker build -t ${imageName} ."
                        sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                        sh "docker push ${imageName}"
                    } 
                } 
            }
    } 
    stage ('Deploy Kubernetes') {
        steps {
            script { 
                def deploymentYaml = """
                apiVersion: apps/v1
                kind: Deployment
                metadata:
                    name: helloservice-deployment
                spec:
                    replicas: 2
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
                              image: ratneshpuskar/${env.JOB_NAME.toLowerCase()}:${env.BUILD_NUMBER}
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
                      nodePort: 30007
                    type: NodePort
                """
                writeFile file: 'deployment.yaml', text: deploymentYaml
                writeFile file: 'service.yaml', text: serviceYaml
                
                sh 'scp -i /var/test.pem deployment.yaml ubuntu@65.2.9.62:~/'
                sh 'scp -i /var/test.pem service.yaml ubuntu@65.2.9.62:~/'
                sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@65.2.9.62 "kubectl apply -f deployment.yaml"'
                sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@65.2.9.62 "kubectl apply -f service.yaml"'
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