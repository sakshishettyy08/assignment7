pipeline {
  agent any

  environment {
    AWS_REGION = 'us-east-1'
    AWS_ACCOUNT_ID = '866934333672'
    ECR_REPO_NAME = 'assignment'
    CLUSTER_NAME = 'assignment-7'
    SERVICE_NAME = 'assignment-service'
    TASK_DEF_NAME = 'assignment-task'
    CONTAINER_NAME = 'assignment-container'
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
          IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
          sh "docker build -t ${IMAGE_URI} ."
        }
      }
    }

    stage('Login to ECR') {
      steps {
        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
      }
    }

    stage('Push Docker Image to ECR') {
      steps {
        sh "docker push ${IMAGE_URI}"
      }
    }

    stage('Fetch Current Task Definition') {
      steps {
        script {
          sh """
            aws ecs describe-task-definition \
              --task-definition ${TASK_DEF_NAME} \
              --region ${AWS_REGION} \
              --query 'taskDefinition' \
              > taskdef.json
          """
        }
      }
    }

    stage('Register New Task Definition Revision') {
      steps {
        script {
          sh """
            cat taskdef.json | jq '
              .containerDefinitions[0].image = "${IMAGE_URI}" |
              del(.status, .revision, .taskDefinitionArn, .requiresAttributes, .compatibilities, .registeredAt, .registeredBy)
            ' > new-taskdef.json

            aws ecs register-task-definition \
              --cli-input-json file://new-taskdef.json \
              --region ${AWS_REGION}
          """
        }
      }
    }

    stage('Update ECS Service') {
      steps {
        sh """
          aws ecs update-service \
            --cluster ${CLUSTER_NAME} \
            --service ${SERVICE_NAME} \
            --task-definition ${TASK_DEF_NAME} \
            --region ${AWS_REGION}
        """
      }
    }
  }
}
