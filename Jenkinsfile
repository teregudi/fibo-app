pipeline {
    agent any

    stages {
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