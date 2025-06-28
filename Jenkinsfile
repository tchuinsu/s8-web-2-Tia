pipeline {
    agent any
    
    triggers {
        githubPush()
    }
    options {
        timeout(time: 30, unit: 'MINUTES') // Timeout for the entire pipeline
        buildDiscarder(logRotator(numToKeepStr: '7')) // Discard old builds to save disk space
        disableConcurrentBuilds() // Ensures that only one build can run at a time
        timestamps() // Adds timestamps to the console output
        skipDefaultCheckout() // Skips the default checkout of source code, useful if you're doing a custom checkout
        retry(3) // Automatically retries the entire pipeline up to 3 times if it fails
    }
    environment {
        DOCKER_HUB_USERNAME="tchuinsu"
        ALPHA_APPLICATION_01_REPO="alpha-application-01"
        ALPHA_APPLICATION_02_REPO="alpha-application-02"
        DOCKER_CREDENTIAL_ID = 'docker-hub-creds'
    }

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 's8coubis1', description: '')
        string(name: 'APP1_TAG', defaultValue: 'latest', description: '')
        string(name: 'APP2_TAG', defaultValue: 'latest', description: '')
        string(name: 'PORT_ON_DOCKER_HOST', defaultValue: '', description: '')
    }

        stages {
            stage('Clone Repository') {
                steps {
                    script {
                        git credentialsId: 'github-auth',
                            url: 'git@github.com:tchuinsu/s8-web-2-Tia.git',
                            branch: "${params.BRANCH_NAME}"
                    }
                }
            }

            stage('Checking the code') {
                steps {
                    script {
                        sh """
                            ls -l
                        """ 
                    }
                }
            }
        }
    }
