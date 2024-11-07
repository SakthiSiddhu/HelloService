pipeline {
    agent any

    tools { 
        maven 'Maven'
    }

    stages {
        stage('Checkout') {
            steps {
                git(branch: 'main', url: 'https://github.com/DatlaBharath/HelloService')
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        def dockerTag = 'ratneshpuskar/helloservice:dockertag'
                        sh "docker build -t ${dockerTag} ."
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        sh "docker push ${dockerTag}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                writeFile file: 'deployment.yaml', text: '''
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
        image: ratneshpuskar/helloservice:dockertag
        ports:
        - containerPort: 5000
'''

                writeFile file: 'service.yaml', text: '''
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
'''

                sshagent(['your-ssh-key-id']) {
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "kubectl apply -f -" < service.yaml'
                    sh 'sleep 60'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "kubectl port-forward --address 0.0.0.0 service/helloservice 5000:5000" || true'
                }
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