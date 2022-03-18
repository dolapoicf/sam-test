pipeline {
  agent any
  environment {
    PIPELINE_USER_CREDENTIAL_ID = 'ga0-hhs'
    S3_BUCKET = 'dolapo-oig'
    SAM_TEMPLATE = 'template.yaml'
    MAIN_BRANCH = 'main'
    TESTING_STACK_NAME = 'dev-app'
    TESTING_PIPELINE_EXECUTION_ROLE = 'arn:aws:iam::318775028588:role/aws-sam-cli-managed-dev-pipe-PipelineExecutionRole-1M2G3SMWM4JLB'
    TESTING_CLOUDFORMATION_EXECUTION_ROLE = 'arn:aws:iam::318775028588:role/aws-sam-cli-managed-dev-p-CloudFormationExecutionR-5FOX47GZWM6H'
    TESTING_ARTIFACTS_BUCKET = 'aws-sam-cli-managed-dev-pipeline-artifactsbucket-gxrlz0xuet5d'
    TESTING_IMAGE_REPOSITORY = '318775028588.dkr.ecr.us-east-1.amazonaws.com/aws-sam-cli-managed-dev-pipeline-resources-imagerepository-2f5dquabctqd'
    TESTING_REGION = 'us-east-1'
    PROD_STACK_NAME = 'stage-app'
    PROD_PIPELINE_EXECUTION_ROLE = 'arn:aws:iam::318775028588:role/aws-sam-cli-managed-stage-pi-PipelineExecutionRole-1BW6CDI4LRDZE'
    PROD_CLOUDFORMATION_EXECUTION_ROLE = 'arn:aws:iam::318775028588:role/aws-sam-cli-managed-stage-CloudFormationExecutionR-TKEE97OKS0C0'
    PROD_ARTIFACTS_BUCKET = 'aws-sam-cli-managed-stage-pipelin-artifactsbucket-pj0x5hunywa2'
    PROD_REGION = 'us-east-1'
  }
  stages {

    stage('build-and-deploy-feature') {
      steps {
        sh 'sam build'
        withAWS(
            credentials: env.PIPELINE_USER_CREDENTIAL_ID,
            region: env.TESTING_REGION,
            role: env.TESTING_PIPELINE_EXECUTION_ROLE,
            roleSessionName: 'deploying-feature') {
          
          sh  'sam deploy --stack-name dolapo-oigx1ab -t template.yaml --s3-bucket $TESTING_ARTIFACTS_BUCKET --capabilities CAPABILITY_IAM'
        }
      }
    }

    stage('build-and-deploy staging env') {
      steps {
        sh 'sam build'
        withAWS(
            credentials: env.PIPELINE_USER_CREDENTIAL_ID,
            region: env.TESTING_REGION,
            role: env.TESTING_PIPELINE_EXECUTION_ROLE,
            roleSessionName: 'testing-packaging') {
          
         // sh 'sam deploy --stack-name dolapo-oigx1ab -t template.yaml --s3-bucket $TESTING_ARTIFACTS_BUCKET --capabilities CAPABILITY_IAM'
        }
      }
       
        }


    stage('deploy-testing') {
      steps {
        withAWS(
            credentials: env.PIPELINE_USER_CREDENTIAL_ID,
            region: env.TESTING_REGION,
            role: env.TESTING_PIPELINE_EXECUTION_ROLE,
            roleSessionName: 'testing-deployment') {
          sh '''
            sam deploy --stack-name ${TESTING_STACK_NAME} \
              --template packaged-testing.yaml \
              --capabilities CAPABILITY_IAM \
              --region ${TESTING_REGION} \
              --s3-bucket ${TESTING_ARTIFACTS_BUCKET} \
              --no-fail-on-empty-changeset \
              --role-arn ${TESTING_CLOUDFORMATION_EXECUTION_ROLE}
          '''
        }
      }
    }


    stage('deploy-prod') {
      steps {
        // uncomment this to have a manual approval step before deployment to production
        // timeout(time: 24, unit: 'HOURS') {
        //   input 'Please confirm before starting production deployment'
        // }
        withAWS(
            credentials: env.PIPELINE_USER_CREDENTIAL_ID,
            region: env.PROD_REGION,
            role: env.PROD_PIPELINE_EXECUTION_ROLE,
            roleSessionName: 'prod-deployment') {
          sh '''
            sam deploy --stack-name ${PROD_STACK_NAME} \
              --template packaged-prod.yaml \
              --capabilities CAPABILITY_IAM \
              --region ${PROD_REGION} \
              --s3-bucket ${PROD_ARTIFACTS_BUCKET} \
              --no-fail-on-empty-changeset \
              --role-arn ${PROD_CLOUDFORMATION_EXECUTION_ROLE}
          '''
        }
      }
    }
  }
}
