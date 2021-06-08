#!/usr/bin/env/ groovy

/*
@author:    Chad Bartel
@date:      2021-06-08
*/


// Begin scripted Jenkins pipeline
pipeline{
    agent{
        node {
            label "docker-agent"
        }
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
                    } else if (env.BRANCH_NAME == 'cert') {
                        // cert params
                    } else if (env.BRANCH_NAME == 'dev') {
                        // dev params
                    } else {
                        // test params
                    }
                    // Print build params
                    // Get role credentials with STS
                    sh '''
                    temp_role=$(aws sts assume-role --role-arn $ROLE_ARN --role-session-name jenkins)
                    echo $temp_role | jq -r .Credentials.AccessKeyId > access_key_id
                    echo $temp_role | jq -r .Credentials.SecretAccessKey > secret_access_key
                    echo $temp_role | jq -r .Credentials.SessionToken > session_token
                    '''
                    // Save role credentials as environment variables
                    env.AWS_ACCESS_KEY_ID = readFile 'access_key_id'
                    env.AWS_SECRET_ACCESS_KEY = readFile 'secret_access_key'
                    env.AWS_SESSION_TOKEN = readFile 'session_token'
                    // Clean up local files
                    sh '''
                    rm access_key_id
                    rm secret_access_key
                    rm session_token
                    '''
                }
            }
        }
        stage("A"){
            steps{
                echo "========executing A========"
            }
            post{
                always{
                    echo "========always========"
                }
                success{
                    echo "========A executed successfully========"
                }
                failure{
                    echo "========A execution failed========"
                }
            }
        }
    }
    post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}