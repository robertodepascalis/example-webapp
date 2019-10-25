def builderImage
def productionImage
def ACCOUNT_REGISTRY_PREFIX
def GIT_COMMIT_HASH

pipeline {
    agent any
    stages {
        stage('Checkout Source code and Logging Into Registry') {
            steps {
                echo 'Logging Into the Private ECR Registry'
                script {
                    GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format: '%H'", returnStdout: true)
                    ACCOUNT_REGISTRY_PREFIX = "480593292995.dkr.ecr.us-east-2.amazonaws.com"
                    sh """
					\$(aws ecr get-login --no-include-email --region us-east-2)
					"""
                }
            }
        }
    }

    stages {
        stage('Make A Builder Image') {
            steps {
                echo 'Start to build the project builder docker image'
                script {
                    builderImage = docker.build("${ACCOUNT_REGISTRY_PREFIX}/example-webapp-builder:${GIT_COMMIT_HASH}", "-f ./Dockerfile.builder .")
                    builderImage.push("{env.GIT_BRANCH}")
                    builderImage.inside('-v $WORKSPACE:/output -u root'){
                        sh """
							cd /output
							lein uberjar
						"""
                    }
                }
            }
        }
    }

    stages {
        stage('Unit Tests') {
            steps {
                echo 'Run unit tests in the builder image'
                script {
                    builderImage = docker.build("${ACCOUNT_REGISTRY_PREFIX}/example-webapp-builder:${GIT_COMMIT_HASH}", "-f ./Dockerfile.builder .")
                    builderImage.push("{env.GIT_BRANCH}")
                    builderImage.inside('-v $WORKSPACE:/output -u root'){
                        sh """
							cd /output
							lein test
						"""
                    }
                }
            }
        }
    }

    stages {
        stage('Build Production Image') {
            steps {
                echo 'Starting to build docker image'
                script {
                    productionImage = docker.build("${ACCOUT_REGISTRY_PREFIX}/example-webapp:${GIT_COMMIT_HASH}", " .")
                    productionImage.push()
                    productionImage.push("{env.GIT_BRANCH}")
                }
            }
        }
    }
}

