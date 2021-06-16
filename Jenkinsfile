#!/usr/bin/env/ groovy

/*
@author:    Chad Bartel
@date:      2021-06-08
*/

def autoCancelled = false


// Begin scripted Jenkins pipeline
pipeline{
  agent{
    node {
      label "docker-agent"
    }
  }
  environment {
    TERRAFORM_CMD = 'docker run --network host " -w /app -v ${HOME}/.aws:/root/.aws -v ${HOME}/.ssh:/root/.ssh -v `pwd`:/app hashicorp/terraform:light'
  }
  triggers {
    // Check repo for commits every five minutes after the hour
    pollSCM('H/5 * * * *')
  }
  stages{
    stage("Set Parameters"){
      steps{
        script {
          // Agnostic parameters
          env.ACCOUNT_HOST = '117580903444'
          if (env.BRANCH_NAME == 'main') {
            // prod params
            env.ENV_VAR = 'prod'
          } else if (env.BRANCH_NAME == 'cert') {
            // dev params
            env.ENV_VAR = 'cert'
          } else if (env.BRANCH_NAME == 'dev') {
            // dev params
            env.ENV_VAR = 'dev'
          } else {
            // test params
            env.ENV_VAR = 'dev'
          }
        }
      }
    }

    stage("Build Terraform light image"){
      steps{
        script {
          sh '''
            docker pull hashicorp/terraform:light
          '''
        }
      }
    }

    stage("Plan Terraform"){
      steps{
        script {
          sh '''
            export ENV_VAR=$(echo $ENV_VAR)
            export TF_VAR_env=$(echo ${ENV_VAR})
            export BACKEND_FILE=backend-${ENV_VAR}.tf
            cp backends/${BACKEND_FILE} backend.tf
            ${TERRAFORM_CMD} init
            ${TERRAFORM_CMD} validate
            ${TERRAFORM_CMD} plan -out=tfplan
          '''
        }
      }
    }

    stage("Get Approval"){
      steps{
        script {
          timeout(time: 10, unit: 'MINUTES'){
            def userInput = input(
              id: 'userInput', 
              message: 'Do you want to continue the pipeline?\nYes or No', 
              parameters: [
                choice(
                  choices: [
                    'Yes', 
                    'No'
                  ], 
                  name: 'approvalInput',
                  description: 'By selecting "Yes" you acknowledge that you reviewed the Terraform plan.'
                )
              ]
            )
            inputApproval = userInput
          }
        }
        if (inputApproval == 'No') {      
          autoCancelled = true
          echo 'Aborted the build'
          error ('Aborting the build as requested')
          currentBuild.result = 'ABORTED'
          return
        }
      }
    }
  }
}