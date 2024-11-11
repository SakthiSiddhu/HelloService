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

        stage('Docker Build & Push') {
            steps {
                script {
                    def image = "ratneshpuskar/helloservice:${env.BUILD_NUMBER}"

                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                        sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                        sh "docker build -t ${image} ."
                        sh "docker push ${image}"
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
    selector:
        app: helloservice
    ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30007
    type: NodePort
"""

                    writeFile(file: 'deployment.yaml', text: deploymentYaml)
                    writeFile(file: 'service.yaml', text: serviceYaml)

                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.194.155 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.194.155 "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful.'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}