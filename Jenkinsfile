pipeline {
    agent any
    
    triggers {
        githubPush()
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '7'))
        disableConcurrentBuilds()
        timestamps()
        skipDefaultCheckout()
        retry(3)
    }

    environment {
        DOCKER_HUB_USERNAME = "tchuinsu"
        ALPHA_APPLICATION_01_REPO = "alpha-application-01"
        ALPHA_APPLICATION_02_REPO = "alpha-application-02"
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
                git credentialsId: 'github-auth',
                    url: 'git@github.com:tchuinsu/s8-web-2-Tia.git',
                    branch: "${params.BRANCH_NAME}"
            }
        }

        stage('Checking the code') {
            steps {
                sh 'ls -l'
                sh 'pwd'
            }
        }
        stage('Building application 01') {
            steps {
                script {
                    sh """
                        pwd
                        ls -l
                        docker build -t ${env.DOCKER_HUB_USERNAME}/app-01:${BUILD_NUMBER} -f application-01.Dockerfile .
                        docker images
                    """ 
                }
            }
        }
        stage('Building application 02') {
            steps {
                script {
                    sh """
                        pwd
                        ls -l
                        docker build -t ${env.DOCKER_HUB_USERNAME}/app-02:${BUILD_NUMBER} -f application-02.Dockerfile .
                        docker images
                    """ 
                }
            }
        }
        stage('Login into') {
            steps {
                script {
                    // Login to Docker Hub
                    withCredentials([usernamePassword(credentialsId: "docker-hub-creds", 
                    usernameVariable: 'DOCKER_USERNAME', 
                    passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Use Docker CLI to login
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    }
                }
            }
        }

        stage('Deploying the application 01') {
            steps {
                script {
                    sh """
                        docker run -itd -p ${params.PORT_ON_DOCKER_HOST_APP_1}:8081  ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_01_REPO}:${params.APP1_TAG}
                        sleep 5
                        docker ps 
                    """ 
                }
            }
        }
        stage('Deploying the application 02') {
            steps {
                script {
                    sh """
                        docker run -itd -p ${params.PORT_ON_DOCKER_HOST_APP_2}:8082  ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_02_REPO}:${params.APP2_TAG}
                        sleep 5
                        docker ps 
                    """ 
                }
            }
        }

    }
}

                        
