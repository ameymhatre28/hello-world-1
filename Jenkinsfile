// Declarative pipelines must be enclosed with a "pipeline" directive.
pipeline {
    // This line is required for declarative pipelines. Just keep it here.
     agent {
        node {
            label'Jenkins-Slave'
        }
    }
     

    // This section contains environment variables which are available for use in the
    // pipeline's stages.
    environment {
 AWS_ACCOUNT_ID="836154072079"
 AWS_DEFAULT_REGION="eu-north-1" 
 IMAGE_REPO_NAME="test"
 IMAGE_TAG="latest"
 REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
 task_def_arn = "arn:aws:ecs:eu-north-1:836154072079:task-definition/Web-Application"
 cluster = "Amey-test"
 exec_role_arn = "arn:aws:iam::836154072079:role/ecsTaskExecutionRole" 
         
 }
 
stages {

         stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }

            }
        }

    
         stage('Code Build') {
            steps {
                sh "mvn clean package" 

            }
        }
      

    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = podman.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }

    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                sh "docker rmi -f ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                sh "docker rmi -f ${IMAGE_REPO_NAME}:${IMAGE_TAG}"
         }
        }
      }
    stage('Deploy') {
    steps {
        // Override image field in taskdef file
        sh "sed -i 's|{{image}}|${REPOSITORY_URI}:${IMAGE_TAG}|' taskdef.json"
        // Create a new task definition revision
        sh "aws ecs register-task-definition --execution-role-arn ${exec_role_arn} --cli-input-json file://taskdef.json --region ${AWS_DEFAULT_REGION}"
        // Update service on Fargate
        sh "aws ecs update-service --cluster ${cluster} --service sample-app-service --task-definition ${task_def_arn} --region ${AWS_DEFAULT_REGION}"
    }
}
    }
}
