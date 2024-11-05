pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        KUBECONFIG = '/home/ec2-user/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/DatlaBharath/HelloService', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = 'ratneshpuskar/helloservice:dockertag'
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                        docker build -t ${imageName} .
                        echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                        docker push ${imageName}
                        """
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
  name: helloservice
spec:
  replicas: 2
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
                sh """
                ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.201.79.94 "kubectl apply -f " < deployment.yaml 
                ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.201.79.94 "kubectl apply -f " < service.yaml
                """
            }
        }

        stage('Sleep') {
            steps {
                script {
                    sleep(60)
                }
            }
        }
        
        stage('Port Forwarding') {
            steps {
                sh """
                ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.201.79.94 "kubectl port-forward --address 0.0.0.0 service/helloservice 5000:5000" &
                sleep 120
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}