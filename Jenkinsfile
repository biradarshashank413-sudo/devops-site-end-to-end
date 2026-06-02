pipeline {
  agent any

  triggers {
    githubPush()
  }

  environment {
    IMAGE = "devops-site"
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main',
            url: 'https://github.com/biradarshashank413-sudo/devops-site-end-to-end.git'
      }
    }

    stage('Build') {
      steps {
        script {
          def VERSION = sh(script: "date +%Y%m%d%H%M", returnStdout: true).trim()
          env.VERSION = VERSION
        }
        sh "docker build -t ${IMAGE}:${env.VERSION} ."
      }
    }

    stage('Login & Push') {
      steps {
        withCredentials([usernamePassword(
              credentialsId: 'DOCKER_CARD',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh """
            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
            docker tag ${IMAGE}:${env.VERSION} \$DOCKER_USER/${IMAGE}:${env.VERSION}
            docker push \$DOCKER_USER/${IMAGE}:${env.VERSION}
          """
        }
      }
    }

    stage('Deploy') {
      steps {
        sh """
          docker stop ${IMAGE} || true
          docker rm   ${IMAGE} || true
          docker run -d \
            --name ${IMAGE} \
            --restart unless-stopped \
            -p 5000:5000 \
            \$DOCKER_USER/${IMAGE}:${env.VERSION}
        """
      }
    }

  }

  post {
    success {
      echo "Build ${env.VERSION} deployed successfully."
    }
    failure {
      echo "Pipeline failed. Check the logs above."
    }
    always {
      sh "docker logout || true"
    }
  }
}
