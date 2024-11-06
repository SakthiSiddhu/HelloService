pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        REPO_URL = 'https://github.com/DatlaBharath/HelloService'
        REPO_BRANCH = 'main'
        DOCKER_REPO = 'ratneshpuskar/helloservice' // project name is 'helloservice'
        DOCKER_TAG = 'dockertag'
        K8S_HOST = '13.201.96.193'
        KEY_PATH = '/var/test.pem'
        SERVICE_NAME = 'helloservice'
    }

    stages {
        stage('Checkout') {
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
                    def appName = 'helloservice' // project name is 'helloservice'
                    sh "docker build -t ${DOCKER_REPO}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Docker Login and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push ${DOCKER_REPO}:${DOCKER_TAG}"
                }
            }
        }

        stage('Create Kubernetes Deployment and Service') {
            steps {
                writeFile file: 'deployment.yaml', text: """
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
        image: ${DOCKER_REPO}:${DOCKER_TAG}
        ports:
        - containerPort: 5000
"""

                writeFile file: 'service.yaml', text: """
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
"""

                sh """
                    ssh -i ${KEY_PATH} -o StrictHostKeyChecking=no ec2-user@${K8S_HOST} "kubectl apply -f -" < deployment.yaml
                    ssh -i ${KEY_PATH} -o StrictHostKeyChecking=no ec2-user@${K8S_HOST} "kubectl apply -f -" < service.yaml
                """

            }
        }

        stage('Port Forward') {
            steps {
                sleep 60
                sh """
                    ssh -i ${KEY_PATH} -o StrictHostKeyChecking=no ec2-user@${K8S_HOST} "kubectl port-forward --address 0.0.0.0 service/${SERVICE_NAME} 5000:5000 &"
                """
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