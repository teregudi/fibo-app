pipeline {
    agent any

    stages {
        stage('Install Docker') {
            steps {
                script {
                    sh '''
                    # Install Docker
                    apt-get update
                    apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common
                    curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
                    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
                    apt-get update
                    apt-get install -y docker-ce-cli
                    '''
                }
            }
        }
        stage('Build and Test Client') {
            steps {
                script {
                    docker.image('node:16').inside {
                        sh 'cd client && npm install && npm test'
                    }
                }
            }
        }
        stage('Build Production Images') {
            steps {
                script {
                    docker.build('teregudi/multi-client', './client')
                    docker.build('teregudi/multi-nginx', './nginx')
                    docker.build('teregudi/multi-server', './server')
                    docker.build('teregudi/multi-worker', './worker')
                }
            }
        }
        stage('Push Images to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhubid') {
                        docker.image('teregudi/multi-client').push()
                        docker.image('teregudi/multi-nginx').push()
                        docker.image('teregudi/multi-server').push()
                        docker.image('teregudi/multi-worker').push()
                    }
                }
            }
        }
    }
}