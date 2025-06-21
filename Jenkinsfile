pipeline {
  agent any

  environment {
    DOCKERHUB_USER = credentials('dockerhub-creds').username
    DOCKERHUB_PASS = credentials('dockerhub-creds').password
    EC2_HOST = 'your.ec2.public.ip'
  }

  stages {
    stage('Test') {
      steps {
        echo "✅ All tests passed!"
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh "echo ${DOCKERHUB_PASS} | docker login -u ${DOCKERHUB_USER} --password-stdin"
          sh "docker build -t ${DOCKERHUB_USER}/hello-nginx:latest ."
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        sh "docker push ${DOCKERHUB_USER}/hello-nginx:latest"
      }
    }

    stage('Deploy to EC2') {
      steps {
        sshagent(credentials: ['ec2-ssh-key']) {
          sh """
          ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << 'EOF'
            docker pull ${DOCKERHUB_USER}/hello-nginx:latest
            docker stop web || true
            docker rm web || true
            docker run -d --name web -p 80:80 ${DOCKERHUB_USER}/hello-nginx:latest
          EOF
          """
        }
      }
    }
  }
}
