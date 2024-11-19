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
                    sh """
                        docker build -t ${imageName} .
                        docker login -u ${env.DOCKERHUB_USER} -p ${env.DOCKERHUB_PASSWORD}
                        docker push ${imageName}
                    """
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
  name: ${env.JOB_NAME.toLowerCase()}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${env.JOB_NAME.toLowerCase()}
  template:
    metadata:
      labels:
        app: ${env.JOB_NAME.toLowerCase()}
    spec:
      containers:
      - name: ${env.JOB_NAME.toLowerCase()}
        image: ratneshpuskar/${env.JOB_NAME.toLowerCase()}:${env.BUILD_NUMBER}
        ports:
        - containerPort: 5000
"""
                  def serviceYaml = """
apiVersion: v1
kind: Service
metadata:
  name: ${env.JOB_NAME.toLowerCase()}
spec:
  type: NodePort
  selector:
    app: ${env.JOB_NAME.toLowerCase()}
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30007
"""
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh """
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.235.113.223 "kubectl apply -f -" < deployment.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.235.113.223 "kubectl apply -f -" < service.yaml
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
