pipeline {
  agent any

  environment {
    AWS_REGION = 'us-east-1'
    ECR_REPO = '866934333672.dkr.ecr.us-east-1.amazonaws.com/assignment'
  }

  options {
    skipDefaultCheckout()
  }

  stages {
    stage('Checkout Alpha Branch') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: '*/alpha']],
          userRemoteConfigs: [[
            url: 'https://github.com/sakshishettyy08/assignment7.git'
          ]]
        ])
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          COMMIT_HASH = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
          IMAGE_TAG = "alpha-${COMMIT_HASH}"
          sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
        }
      }
    }

    stage('Authenticate with ECR') {
      steps {
        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
      }
    }

    stage('Push Docker Image to ECR') {
      steps {
        script {
          sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
        }
      }
    }

    stage('Trigger CD Pipeline') {
      steps {
        build job: 'CD-Pipeline-Deploy-to-ECS', parameters: [
          string(name: 'IMAGE_TAG', value: "${IMAGE_TAG}")
        ]
      }
    }
  }
}
