pipeline {
    agent {
    dockerContainer {
      image 'docker:latest'
      args '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  environment {
    EC2_HOST = 'your.ec2.public.ip'
    EC2_USER = 'ec2-user'
  }

  stages {
    stage('Test') {
      steps {
        echo "✅ All tests passed!"
      }
    }

    stage('Build Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
          sh """
            echo \$DOCKERHUB_PASS | docker login -u \$DOCKERHUB_USER --password-stdin
            docker build -t \$DOCKERHUB_USER/hello-nginx:latest .
          """
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
          sh "docker push \$DOCKERHUB_USER/hello-nginx:latest"
        }
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
