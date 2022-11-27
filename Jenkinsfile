pipeline {
  agent any
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
   
     stage('Build Docker Image') {
       agent{label 'docker_agent'}
          steps {
            withCredentials([getSecret(credentialsId: 'DOCKERHUB_ACCESS_KEY', variable: 'dockerhub_access_key'), 
                            getSecret(credentialsId: 'DOCKERHUB_SECRET_KEY', variable: 'dockerhub_secret_key')]) {
                                  sh '''#!/bin/bash
                                  sudo curl https://github.com/AdreReyes/Docker-Terraform_Deployment5/blob/main/dockerfile > dockerfile
                                  sudo docker login --username=${dockerhub_access_key} --password=${dockerhub_secret_key}
                                  sudo docker build -t deploy5-image:latest .
                                  sudo docker image push adrereyes1/flask-app:latest
                                  '''
                                }
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
      stage('Apply') {
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
