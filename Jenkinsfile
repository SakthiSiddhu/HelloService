pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub_credentials')
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/DatlaBharath/HelloService']]])
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
                    def projectName = "helloservice" // Extracted based on given GitHub URL
                    def dockertag = "latest"
                    def dockerImage = "${env.DOCKER_CREDENTIALS_USR}/${projectName}:${dockertag}"

                    sh "docker build -t ${dockerImage} ."

                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh "echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin"
                        sh "docker push ${dockerImage}"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYaml = """\
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloservice-deployment
  labels:
    app: helloservice
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
        image: ${env.DOCKER_CREDENTIALS_USR}/helloservice:latest
        ports:
        - containerPort: 5000
                    """.stripIndent()

                    def serviceYaml = """\
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
                    """.stripIndent()

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh '''
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@43.205.240.201 "kubectl apply -f -" < deployment.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@43.205.240.201 "kubectl apply -f -" < service.yaml
                        sleep 60
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@43.205.240.201 "kubectl port-forward --address 0.0.0.0 service/helloservice 5000:5000"
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Job completed successfully!'
        }
        failure {
            echo 'Job failed!'
        }
    }
}