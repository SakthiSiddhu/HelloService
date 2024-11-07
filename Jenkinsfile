pipeline {
    agent any
    
    tools {
        maven 'Maven'
    }
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
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
                    def dockerImage = "ratneshpuskar/helloservice:dockertag"
                    sh "docker build -t ${dockerImage} ."
                    sh "echo ${env.DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${env.DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    sh "docker push ${dockerImage}"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                writeFile(file: 'deployment.yaml', text: '''
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloservice-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      run: helloservice
  template:
    metadata:
      labels:
        run: helloservice
    spec:
      containers:
      - name: helloservice
        image: ratneshpuskar/helloservice:dockertag
        ports:
        - containerPort: 5000
''')
                
                writeFile(file: 'service.yaml', text: '''
apiVersion: v1
kind: Service
metadata:
  name: helloservice-service
spec:
  type: NodePort
  selector:
    run: helloservice
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
''')

                script {
                    sh '''
ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "kubectl apply -f -" < deployment.yaml
ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "kubectl apply -f -" < service.yaml
sleep 60
ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true
ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@52.66.179.41 "kubectl port-forward --address 0.0.0.0 service/helloservice-service 5000:5000" || true
'''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Job completed successfully.'
        }
        failure {
            echo 'Job failed.'
        }
    }
}