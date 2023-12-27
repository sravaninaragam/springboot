pipeline {
    agent any

    environment {
        mvnHome = tool 'Maven3'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'mvn -f MyAwesomeApp/pom.xml clean install'
                }
            }
        }

        stage('Archive') {
            steps {
                script {
                    archiveArtifacts '**/*.jar'
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    // Build and push image with Jenkins' docker-plugin
                    withDockerServer([uri: "tcp://localhost:4243"]) {
                        withDockerRegistry([credentialsId: "fa32f95a-2d3e-4c7b-8f34-11bcc0191d70", url: "https://index.docker.io/v1/"]) {
                            def customImage = docker.build("ananthkannan/mywebapp", "MyAwesomeApp")
                            customImage.push()
                        }
                    }
                }
            }
        }

        stage('Docker Stop Container') {
            steps {
                script {
                    sh 'docker ps -f name=myContainer -q | xargs --no-run-if-empty docker container stop'
                    sh 'docker container ls -a -fname=myContainer -q | xargs -r docker container rm'
                }
            }
        }

        stage('Docker Run') {
            steps {
                script {
                    customImage = docker.image("ananthkannan/mywebapp")
                    customImage.run("-p 8085:8085 --rm --name myContainer")
                }
            }
        }
    }
}
