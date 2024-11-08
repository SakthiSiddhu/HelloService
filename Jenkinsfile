pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        DOCKER_REPO = 'ratneshpuskar'
        DOCKER_CREDENTIALS_ID = 'dockerhub_credentials'
        DOCKER_IMAGE_TAG = "latest"
        DOCKER_PROJECT_NAME = "helloservice"
        DEPLOYMENT_FILE = 'deployment.yaml'
        SERVICE_FILE = 'service.yaml'
        EC2_USER = 'ec2-user'
        EC2_HOST = '13.201.20.4'
        KEY_FILE = '/var/test.pem'
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
        
        stage('Docker Build') {
            steps {
                script {
                    def projectName = 'helloservice'.toLowerCase()
                    def imageTag = "${DOCKER_REPO}/${projectName}:${DOCKER_IMAGE_TAG}"
                    sh "docker build -t ${imageTag} ."
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    script {
                        def projectName = 'helloservice'.toLowerCase()
                        def imageTag = "${DOCKER_REPO}/${projectName}:${DOCKER_IMAGE_TAG}"
                        sh "docker push ${imageTag}"
                    }
                }
            }
        }
        
        stage('Kubernetes Deploy') {
            steps {
                script {
                    def deploymentFileContent = '''
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
        image: ratneshpuskar/helloservice:latest
        ports:
        - containerPort: 5000
                    '''

                    def serviceFileContent = '''
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
  type: NodePort
                    '''
                    
                    writeFile file: 'deployment.yaml', text: deploymentFileContent
                    writeFile file: 'service.yaml', text: serviceFileContent

                    sh "scp -i ${KEY_FILE} -o StrictHostKeyChecking=no ${DEPLOYMENT_FILE} ${EC2_USER}@${EC2_HOST}:${DEPLOYMENT_FILE}"
                    sh "scp -i ${KEY_FILE} -o StrictHostKeyChecking=no ${SERVICE_FILE} ${EC2_USER}@${EC2_HOST}:${SERVICE_FILE}"

                    sh '''
                    ssh -i ${KEY_FILE} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "kubectl apply -f ${DEPLOYMENT_FILE}"
                    ssh -i ${KEY_FILE} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "kubectl apply -f ${SERVICE_FILE}"
                    sleep 60
                    ssh -i ${KEY_FILE} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "sudo lsof -t -i :5000 | xargs -r sudo kill -9 || true"
                    ssh -i ${KEY_FILE} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000 || true"
                    '''
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