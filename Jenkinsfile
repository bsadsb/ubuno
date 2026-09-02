pipeline {
    agent { label 'docker' }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: '1.0.0', description: 'Tag to use with the new image version')
        booleanParam(name: 'SET_IMAGE_LATEST', defaultValue: false, description: "Move the 'latest' image tag to the newly built version")
    }

    environment {
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub'
        IMAGE                    = 'bitbar/ubuno'
        TAG                      = "${params.IMAGE_TAG.trim()}"
    }

    options {
        ansiColor('xterm')
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Tag Presence') {
            steps {
                script {
                    def exists = sh(script: "docker manifest inspect ${IMAGE}:${TAG} > /dev/null 2>&1", returnStatus: true) == 0
                    if (exists) {
                        error("${IMAGE}:${TAG} already exists on the registry. Bump IMAGE_TAG to avoid overwriting an existing image.")
                    }
                }
            }
        }

        stage('Build & Push') {
            steps {
                script {
                    def newImage = docker.build("${IMAGE}:${TAG}", "--pull .")

                    docker.withRegistry('https://registry.hub.docker.com', DOCKERHUB_CREDENTIALS_ID) {
                        newImage.push()
                        if (params.SET_IMAGE_LATEST) {
                            newImage.push('latest')
                        }
                    }

                    sh "docker rmi ${IMAGE}:${TAG}"
                }
            }
        }
    }

    post {
        success {
            echo "Successfully pushed ${IMAGE}:${TAG}"
        }
        failure {
            echo "Build/push failed for ${IMAGE}:${TAG}"
        }
    }
}
