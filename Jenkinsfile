pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    ECR_REPO   = "demo-nodejs-app"
    IMAGE_TAG  = "${ECR_REPO}:${BUILD_NUMBER}"
    ECR_URI    = "123456789012.dkr.ecr.ap-south-1.amazonaws.com/${ECR_REPO}"
  }

  stages {

    stage('Checkout Source') {
  steps {
    git branch: 'main', url: 'https://github.com/Boyshiva03/My-Project.git'
  }
}

    stage('Login to ECR') {
      steps {
        script {
          sh """
            aws ecr get-login-password --region ${AWS_REGION} | \
            docker login --username AWS --password-stdin ${ECR_URI}
          """
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${ECR_URI}:${BUILD_NUMBER} ."
      }
    }

    stage('Push Docker Image to ECR') {
      steps {
        sh "docker push ${ECR_URI}:${BUILD_NUMBER}"
      }
    }

    stage('Deploy to EKS') {
      steps {
        sh """
          sed -i 's|image:.*|image: ${ECR_URI}:${BUILD_NUMBER}|' k8s/deployment.yaml
          kubectl apply -f k8s/deployment.yaml
          kubectl apply -f k8s/service.yaml
        """
      }
    }
  }

  post {
    success {
      echo "✅ Deployment completed!"
    }
    failure {
      echo "❌ Something went wrong. Check logs."
    }
  }
}
