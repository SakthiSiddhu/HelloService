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
                    def projectname = 'helloservice'
                    def dockerTag = "latest"
                    docker.build("ratneshpuskar/${projectname.toLowerCase()}:${dockerTag}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    sh 'echo $DOCKER_HUB_PASSWORD | docker login --username $DOCKER_HUB_USERNAME --password-stdin'
                    def projectname = 'helloservice'
                    def dockerTag = "latest"
                    sh 'docker push ratneshpuskar/${projectname.toLowerCase()}:${dockerTag}'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def projectname = 'helloservice'
                    def dockerTag = "latest"
                    sh '''cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $projectname-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: $projectname
  template:
    metadata:
      labels:
        app: $projectname
    spec:
      containers:
      - name: $projectname
        image: ratneshpuskar/$projectname:${dockerTag}
        ports:
        - containerPort: 5000
EOF
                    '''

                    sh '''cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: $projectname-service
spec:
  type: NodePort
  ports:
  - port: 5000
    targetPort: 5000
  selector:
    app: $projectname
EOF
                    '''

                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
}