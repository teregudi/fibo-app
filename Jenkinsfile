pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

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
        stage('Deploy to Elastic Beanstalk') {
            steps {
                script {
                    sh '''
                    # Configure AWS CLI
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set default.region eu-north-1

                    # Upload Docker images to S3
                    aws s3 cp ./Dockerrun.aws.json s3://elasticbeanstalk-eu-north-1-904233114398/fibo-app/Dockerrun.aws.json

                    # Initialize Elastic Beanstalk application
                    eb init -p docker fibo-app --region eu-north-1

                    # Create Elastic Beanstalk environment if it doesn't exist
                    if ! eb status faszom-env; then
                        eb create faszom-env
                    fi

                    # Deploy to Elastic Beanstalk
                    eb deploy faszom-env
                    '''
                }
            }
        }
    }
}