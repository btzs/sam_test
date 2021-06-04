pipeline {
  agent any
 
  stages {
    stage('Install sam-cli') {
      steps {
        sh 'python3 -m venv venv && venv/bin/pip install aws-sam-cli'
        stash includes: '**/venv/**/*', name: 'venv'
      }
    }
    stage('Build') {
      steps {
        unstash 'venv'
        sh 'venv/bin/sam build'
        stash includes: '**/.aws-sam/**/*', name: 'aws-sam'
      }
    }
    stage('beta') {
      environment {
        STACK_NAME = 'sam-app-beta-stage'
        S3_BUCKET = 'sam-jenkins-arpm-dev-2021-06-04'
      }
      steps {
        withAWS(credentials: 'sam-jenkins-demo-credentials', region: 'ap-southeast-1') {
          unstash 'venv'
          unstash 'aws-sam'
          sh 'ls'
          sh 'pwd'
          sh 'cat template.yml'
          sh 'cat .aws-sam/build/template.yaml'
          sh 'venv/bin/sam deploy --stack-name $STACK_NAME -t .aws-sam/build/template.yaml --s3-bucket $S3_BUCKET --capabilities CAPABILITY_IAM'
          dir ('hello-world') {
            sh 'npm ci'
            sh 'npm run integ-test'
          }
        }
      }
    }
    stage('prod') {
      environment {
        STACK_NAME = 'sam-app-prod-stage'
        S3_BUCKET = 'sam-jenkins-arpm-prod-2021-06-04'
      }
      steps {
        withAWS(credentials: 'sam-jenkins', region: 'ap-southeast-1') {
          unstash 'venv'
          unstash 'aws-sam'
          sh 'venv/bin/sam deploy --stack-name $STACK_NAME -t template.yaml --s3-bucket $S3_BUCKET --capabilities CAPABILITY_IAM'
        }
      }
    }
  }
}