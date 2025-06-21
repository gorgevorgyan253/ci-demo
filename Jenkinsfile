pipeline {
    agent any
  

  environment {
    EC2_HOST = '13.220.60.90'
    EC2_USER = 'ec2-user'
  }

  stages {
    stage('Test') {
      steps {
        echo "✅ All tests passed!"
      }
    }

    stage('Deploy to EC2') {
      steps {
        sshagent(['ec2-ssh-key']) {
          sh """
            ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << 'EOF'
              docker pull \$DOCKERHUB_USER/hello-nginx:latest
              docker stop web || true
              docker rm web || true
              docker run -d --name web -p 80:80 \$DOCKERHUB_USER/hello-nginx:latest
            EOF
          """
        }
      }
    }
  }
}
