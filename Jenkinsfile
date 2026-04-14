pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    parameters {
        choice(name: 'ENV', choices: ['dev', 'staging'], description: 'Environment')
        string(name: 'VERSION', defaultValue: 'v1.0', description: 'App version')
    }
    
    stages {
        stage('Code Checkout'){
            steps {
                git branch: 'main', url: 'https://github.com/dakshsawhneyy/End2End-CICD-Jenkins.git'
            }
        }
        stage('Install Dependencies with caching') {
            steps {
                sh 'npm install --cache .npm'
            }
        }
        stage('Test') {
            steps {
                echo "Running Tests"
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t myapp:${VERSION} .'
            }
        }
        stage('Push to Dockerhub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag myapp:${VERSION} $DOCKER_USER/jenkinsnodeapp:${VERSION}
                        docker push $DOCKER_USER/jenkinsnodeapp:${VERSION}
                    '''
                }
            }
        }
        stage('Deploy') {
            when {
                    expression { params.ENV == 'staging' }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh  '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker pull $DOCKER_USER/jenkinsnodeapp:${VERSION}
                        docker run -d -p 3000:3000 $DOCKER_USER/jenkinsnodeapp:${VERSION}
                    '''
                }
            }
        }
    }
}
