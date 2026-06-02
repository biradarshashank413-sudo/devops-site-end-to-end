pipeline {
  agent any

  triggers {
    githubPush()
  }

  environment {
    IMAGE = "devops-site"
    GCP_PROJECT = "jenkins-doc"
    GKE_CLUSTER = "cluster-jenkins"
    GKE_ZONE = "us-central1-c"
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
          env.VERSION = sh(script: "date +%Y%m%d%H%M", returnStdout: true).trim()
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
            docker tag ${IMAGE}:${env.VERSION} \$DOCKER_USER/${IMAGE}:latest

            docker push \$DOCKER_USER/${IMAGE}:${env.VERSION}
            docker push \$DOCKER_USER/${IMAGE}:latest

            echo "\$DOCKER_USER/${IMAGE}:${env.VERSION}" > image.txt
          """
        }
      }
    }

    stage('Deploy to GKE') {
      steps {
        withCredentials([
          file(credentialsId: '113962110009254531075', variable: 'GCP_KEY'),
          usernamePassword(
            credentialsId: 'DOCKER_CARD',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
          )
        ]) {
          sh """
            gcloud auth activate-service-account --key-file=\$GCP_KEY
            gcloud config set project ${GCP_PROJECT}

            gcloud container clusters get-credentials ${GKE_CLUSTER} \
              --zone ${GKE_ZONE} \
              --project ${GCP_PROJECT}

            kubectl apply -f k8s/deployment.yaml
            kubectl apply -f k8s/service.yaml

            kubectl set image deployment/flask-app \
              flask-app=\$DOCKER_USER/${IMAGE}:${env.VERSION}

            kubectl rollout status deployment/flask-app
          """
        }
      }
    }
  }

  post {
    success {
      echo "Build ${env.VERSION} deployed successfully to GKE."
    }

    failure {
      echo "Pipeline failed. Check the logs above."
    }

    always {
      sh "docker logout || true"
    }
  }
}