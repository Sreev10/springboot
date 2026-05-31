pipeline {

    environment {
        DOCKER_IMAGE = "sreevastava/springboot-demo"
        MINIKUBE_SERVER = "ec2-user@3.237.64.154"
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/Sreev10/springboot.git'
            }
        }

        stage('Build Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $DOCKER_IMAGE:latest'
            }
        }

        stage('Deploy to Minikube') {
    steps {

        sshagent(['minikube-ssh']) {

            sh '''
            scp -o StrictHostKeyChecking=no \
            k8s/*.yaml \
            ec2-user@3.237.64.154:/home/ec2-user/k8s/

            ssh -o StrictHostKeyChecking=no \
            ec2-user@3.237.64.154 << EOF

            kubectl apply -f /home/ec2-user/k8s/deployment.yaml
            kubectl apply -f /home/ec2-user/k8s/service.yaml

            kubectl rollout restart deployment springboot-demo

            EOF
            '''
        }
    }
}
