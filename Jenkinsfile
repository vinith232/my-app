pipeline {
  agent any

  environment {
    AWS_REGION = 'ap-south-1'
    ECR_REPO = '884394270671.dkr.ecr.ap-south-1.amazonaws.com/my-app'
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout Code') {
      steps {
        git url: 'https://github.com/your-username/my-app.git', branch: 'main'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t my-app .'
        sh 'docker tag my-app:latest $ECR_REPO:$IMAGE_TAG'
      }
    }

    stage('Login to ECR') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred']]) {
          sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO'
        }
      }
    }

    stage('Push to ECR') {
      steps {
        sh 'docker push $ECR_REPO:$IMAGE_TAG'
      }
    }

    stage('Deploy to ECS') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred']]) {
          sh '''
          sed -e "s|<IMAGE>|$ECR_REPO:$IMAGE_TAG|" deploy/ecs-task-def.json > task-def-updated.json

          aws ecs register-task-definition \
            --cli-input-json file://task-def-updated.json

          aws ecs update-service \
            --cluster my-cluster \
            --service my-app-service \
            --force-new-deployment
          '''
        }
      }
    }
  }
}
