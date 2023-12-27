pipeline {
    agent any

    environment {
        registry = "912583867239.dkr.ecr.us-west-1.amazonaws.com/my-docker-repo"
        sonarqubeScannerHome = tool 'sonar-scanner'
    }

    stages {
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/sravaninaragam/springboot.git']]])
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build registry
                }
            }
        }

        stage('Pushing to ECR') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: 'AWS']]) {
                        sh '''
                            aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin 912583867239.dkr.ecr.us-west-1.amazonaws.com
                            docker push 912583867239.dkr.ecr.us-west-1.amazonaws.com/my-docker-repo:latest
                            aws eks update-kubeconfig --region us-west-1 --name k8s-eks
                            kubectl get nodes
                            pwd
                            ls
                            kubectl apply -f eks-deploy-k8s.yaml
                        '''
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=sonar -Dsonar.sources=. -Dsonar.host.url=http://3.101.125.242:9000"
                }
            }
        }

        stage('Push to JFrog Artifactory') {
            steps {
                sh 'jfrog --version' 
                sh 'jfrog rt c clear'
                sh 'jfrog rt ping --url=http://54.190.58.123:8082/artifactory'
                sh 'jfrog rt c --url=http://54.190.58.123:8082/artifactory --user=developer --password=Developer@123'
                sh 'jfrog rt c show'
                sh 'jfrog rt u target/*.jar my-repo'
            }
        }

        stage('vault secrets') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'vault-secrets', variable: 'SECRET_KEY')]) {
                        // Your pipeline steps using the secret
                        sh 'echo $SECRET_KEY'
                    }
                }
            }
        }
    }
}
