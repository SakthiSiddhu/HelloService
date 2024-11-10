pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        REPO_URL = 'https://github.com/DatlaBharath/HelloService'
        REPO_BRANCH = 'main'
        DOCKER_CREDENTIALS = 'dockerhub_credentials'
        DOCKER_IMAGE_NAME = 'ratneshpuskar/helloservice'
        KUBERNETES_HOST = 'ubuntu@43.205.198.61'
        SSH_KEY_PATH = '/var/test.pem'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "${REPO_BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}'
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
        image: ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}
        ports:
        - containerPort: 5000
"""

                    def serviceYaml = """
apiVersion: v1
kind: Service
metadata:
  name: helloservice-service
spec:
  type: NodePort
  selector:
    app: helloservice
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30007
"""

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh """
                    ssh -i ${SSH_KEY_PATH} -o StrictHostKeyChecking=no ${KUBERNETES_HOST} "kubectl apply -f -" < deployment.yaml
                    ssh -i ${SSH_KEY_PATH} -o StrictHostKeyChecking=no ${KUBERNETES_HOST} "kubectl apply -f -" < service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful'
        }

        failure {
            echo 'Deployment failed'
        }
    }
}