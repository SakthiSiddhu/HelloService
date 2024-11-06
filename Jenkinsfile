pipeline {
    agent any
    tools {
        maven "Maven"
    }
    environment {
        REPO_URL = 'https://github.com/DatlaBharath/HelloService'
        BRANCH = 'main'
        DOCKER_CREDENTIALS_ID = 'dockerhub_credentials'
        DOCKER_REPO = 'ratneshpuskar'
        DOCKER_TAG = 'dockertag'
        K8S_USER = 'ec2-user'
        K8S_HOST = '13.235.128.206'
        K8S_KEY = '/var/test.pem'
        DOCKER_IMAGE = "${DOCKER_REPO}/helloservice:${DOCKER_TAG}"
        SERVICE_NAME = 'service-name'
        SERVICE_PORT = 5000
        CONTAINER_PORT = 5000
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: BRANCH]],
                          userRemoteConfigs: [[url: REPO_URL]]])
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
                    docker.build(DOCKER_IMAGE)
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deployment = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${SERVICE_NAME}-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${SERVICE_NAME}
  template:
    metadata:
      labels:
        app: ${SERVICE_NAME}
    spec:
      containers:
      - name: ${SERVICE_NAME}
        image: ${DOCKER_IMAGE}
        ports:
        - containerPort: ${CONTAINER_PORT}
"""
                    def service = """
apiVersion: v1
kind: Service
metadata:
  name: ${SERVICE_NAME}
spec:
  selector:
    app: ${SERVICE_NAME}
  ports:
    - protocol: TCP
      port: ${SERVICE_PORT}
      targetPort: ${CONTAINER_PORT}
"""
                    sh """
echo "${deployment}" > deployment.yaml
echo "${service}" > service.yaml
ssh -i ${K8S_KEY} -o StrictHostKeyChecking=no ${K8S_USER}@${K8S_HOST} "kubectl apply -f -" < deployment.yaml
ssh -i ${K8S_KEY} -o StrictHostKeyChecking=no ${K8S_USER}@${K8S_HOST} "kubectl apply -f -" < service.yaml
sleep 60
ssh -i ${K8S_KEY} -o StrictHostKeyChecking=no ${K8S_USER}@${K8S_HOST} "sudo lsof -t -i :${SERVICE_PORT} | xargs -r sudo kill -9" || true
ssh -i ${K8S_KEY} -o StrictHostKeyChecking=no ${K8S_USER}@${K8S_HOST} "kubectl port-forward --address 0.0.0.0 service/${SERVICE_NAME} ${CONTAINER_PORT}:${SERVICE_PORT}"
"""
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully.'
        }
        failure {
            echo 'Pipeline failed to execute.'
        }
    }
}