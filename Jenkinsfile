pipeline {
    agent { label 'slave1' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerloginid')
    }

    stages {
        stage('SCM Checkout') {
            steps {
                git 'https://github.com/AbhishekPande285/star-agile-health-care.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t abhishekpande285/medicure-app:${BUILD_NUMBER} .'
                sh 'docker tag abhishekpande285/medicure-app:${BUILD_NUMBER} abhishekpande285/medicure-app:latest'
            }
        }

        stage('DockerHub Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push abhishekpande285/medicure-app:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: 'Kubernetes_Master',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: '*.yaml',
                                    remoteDirectory: '.',
                                    execCommand: 'kubectl apply -f kubernetesdeploy.yaml',
                                    execTimeout: 120000
                                )
                            ],
                            useWorkspaceInPromotion: true,
                            verbose: true
                        )
                    ])
                }
            }
        }
    }
}
