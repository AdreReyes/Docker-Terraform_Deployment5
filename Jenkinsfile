pipeline {
  agent any
  environment {
    DOCKERHUB_CREDENTIALS=credentials('dockerhub')
   stages {
    stage ('Build') {
      steps {
        sh '''#!/bin/bash
        python3 -m venv test3
        source test3/bin/activate
        pip install pip --upgrade
        pip install -r requirements.txt
        export FLASK_APP=application
        flask run &
        '''
     }
   }
    stage ('Test') {
      steps {
        sh '''#!/bin/bash
        source test3/bin/activate
        py.test --verbose --junit-xml test-reports/results.xml
        ''' 
      }
    
      post{
        always {
          junit 'test-reports/results.xml'
        }
       
      }
    }
     stage ('Create Container') {
        agent { label 'docker_agent' }
        steps {
          sh 'docker build -t imagedp5:v1.0'
        }
      }
     
     stage('Push to Dockerhub') {
       agent{label 'docker_agent'}
          steps {
                  sh '''#!/bin/bash
                  docker tag imagedp5:v1.0 adrereyes1/flask-app:v1.0
                  echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
                  docker push adrereyes1/flask-app:v1.0
                  '''
                }
            }

      stage('Init') {
        agent{label 'terraform_agent'}
          steps {
            withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'), 
                            string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
                                dir('intTerraform') {
                                  sh 'terraform init' 
                                }
            }
        }
      }

      stage('Plan') {
       agent{label 'terraform_agent'}
          steps {
            withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'), 
                            string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
                                dir('intTerraform') {
                                  sh 'terraform plan -out plan.tfplan -var="aws_access_key=$aws_access_key" -var="aws_secret_key=$aws_secret_key"' 
                                }
            }
        }
      }

      stage('Deploy to ECS') {
        agent{label 'terraform_agent'}
          steps {
            withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'), 
                            string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
                                dir('intTerraform') {
                                  sh 'terraform apply plan.tfplan' 
                                }
            }
        }
      }
   }
  }
}
