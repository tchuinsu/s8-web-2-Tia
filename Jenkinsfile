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
        string(name: 'BRANCH_NAME', defaultValue: 's8coubis1', description: 'Git branch to build')
        string(name: 'APP1_TAG', defaultValue: 'latest', description: 'Tag for Application 01')
        string(name: 'APP2_TAG', defaultValue: 'latest', description: 'Tag for Application 02')
        string(name: 'PORT_APP1', defaultValue: '8083', description: 'Host port for App 01')
        string(name: 'PORT_APP2', defaultValue: '8084', description: 'Host port for App 02')
        string(name: 'PORT_ON_DOCKER_HOST', defaultValue: '', description: 'Port on Docker host')
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${params.BRANCH_NAME}"]],
                    userRemoteConfigs: [[
                        credentialsId: 'github-auth',
                        url: 'git@github.com:tchuinsu/s8-web-2-Tia.git'
                    ]]
                ])
            }
        }

        stage('Check Code Structure') {
            steps {
                sh 'ls -l'
                sh 'pwd'
            }
        }

        stage('Build Application 01') {
            steps {
                sh """
                    docker build -t ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_01_REPO}:${params.APP1_TAG} -f application-01.Dockerfile .
                    docker images
                """
            }
        }

        stage('Build Application 02') {
            steps {
                sh """
                    docker build -t ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_02_REPO}:${params.APP2_TAG} -f application-02.Dockerfile .
                    docker images
                """
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKER_CREDENTIAL_ID,
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }
            }
        }

        stage('Push Application 01 to DockerHub') {
            steps {
                sh "docker push ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_01_REPO}:${params.APP1_TAG}"
            }
        }

        stage('Push Application 02 to DockerHub') {
            steps {
                sh "docker push ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_02_REPO}:${params.APP2_TAG}"
            }
        }

        stage('Deploy Application 01') {
            steps {
                sh """
                    docker ps -q --filter "publish=${params.PORT_APP1}" | xargs -r docker stop
                    docker ps -a -q --filter "publish=${params.PORT_APP1}" | xargs -r docker rm
                    docker run -itd --name app01 -p ${params.PORT_APP1}:80 ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_01_REPO}:${params.APP1_TAG}
                    sleep 5
                    docker ps
                """
            }
        }


        stage('Deploy Application 02') {
            steps {
                sh """
                    docker ps -q --filter "publish=${params.PORT_APP2}" | xargs -r docker stop
                    docker ps -a -q --filter "publish=${params.PORT_APP2}" | xargs -r docker rm
                    docker run -itd --name app02 -p ${params.PORT_APP2}:80 ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_02_REPO}:${params.APP2_TAG}
                    sleep 5
                    docker ps
                """
            }
        }
    }
}
