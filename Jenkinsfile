pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        kubernetesConfig = '/home/user/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/DatlaBharath/HelloService'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    def appName = 'helloservice'
                    def dockerImage = "ratneshpuskar/${appName.toLowerCase()}:dockerTag"

                    sh "docker build -t ${dockerImage} ."

                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        sh "docker push ${dockerImage}"
                        sh "docker logout"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def appName = 'helloservice'
                    def dockerImage = "ratneshpuskar/${appName.toLowerCase()}:dockerTag"
                    def deploymentYaml = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${appName}-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${appName}
  template:
    metadata:
      labels:
        app: ${appName}
    spec:
      containers:
      - name: ${appName}
        image: ${dockerImage}
        ports:
        - containerPort: 5000
"""
                    def serviceYaml = """
apiVersion: v1
kind: Service
metadata:
  name: ${appName}-service
spec:
  type: NodePort
  ports:
    - port: 5000
      targetPort: 5000
      nodePort: 30000
  selector:
    app: ${appName}
"""

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    withEnv(['KUBECONFIG=${env.kubernetesConfig}']) {
                        sh 'kubectl apply -f deployment.yaml'
                        sh 'kubectl apply -f service.yaml'
                    }
                }
            }
        }
    }
}