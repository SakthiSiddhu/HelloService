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

        stage('Docker Build and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        def dockerImage = "${DOCKER_USERNAME}/helloservice:latest".toLowerCase()
                        sh "docker build -t ${dockerImage} ."
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                        sh "docker push ${dockerImage}"
                    }
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
                      app: helloservice
                  template:
                    metadata:
                      labels:
                        app: helloservice
                    spec:
                      containers:
                      - name: helloservice
                        image: docker_username/helloservice:latest
                        ports:
                        - containerPort: 5000
                ''')

                writeFile(file: 'service.yaml', text: '''
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
                ''')

                sh '''
                ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < deployment.yaml
                ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl apply -f -" < service.yaml
                sleep 60
                ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "sudo lsof -t -i :5000 | xargs -r sudo kill -9" || true
                ssh -i /var/test.pem -o StrictHostKeyChecking=no ec2-user@13.235.128.206 "kubectl port-forward --address 0.0.0.0 service/helloservice 5000:5000"
                '''
            }
        }
    }

    post {
        success {
            sh 'echo "Pipeline executed successfully."'
        }
        failure {
            sh 'echo "Pipeline execution failed."'
        }
    }
}