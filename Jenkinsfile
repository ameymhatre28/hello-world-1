// Declarative pipelines must be enclosed with a "pipeline" directive.
pipeline {
    // This line is required for declarative pipelines. Just keep it here.
     agent {
        node {

        label 'Jenkins-Slave'

        }

    }

    // This section contains environment variables which are available for use in the
    // pipeline's stages.
    environment {
 AWS_ACCOUNT_ID="048134285155"
 AWS_DEFAULT_REGION="ap-southeast-1" 
 IMAGE_REPO_NAME="test"
 IMAGE_TAG="latest"
 REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
 task_def_arn = "arn:aws:ecs:ap-southeast-1:048134285155:task-definition/Jenkins"
 cluster = "Amey-test"
 exec_role_arn = "arn:aws:iam::048134285155:role/ecsTaskExecutionRole" 
         
 }
 
stages {

         stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }

            }
        }

        //stage('Cloning Git') {
            //steps {
                //checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/sd031/aws_codebuild_codedeploy_nodeJs_demo.git']]])     
            //}
        //}

    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
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
