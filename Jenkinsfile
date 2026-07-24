pipeline {
    agent any

    stages {

        stage('Sonar Analysis') {
            steps {
                echo "LMS code analysis"
                sh '''
                cd webapp
                docker run --rm \
                -e SONAR_HOST_URL="http://18.119.19.122:9000" \
                -e SONAR_TOKEN="sqp_73a9923922cb60694f9c14924872baf3b59d781f" \
                -v "$PWD:/usr/src" \
                sonarsource/sonar-scanner-cli \
                -Dsonar.projectKey=LMS-Project \
                -Dsonar.projectName="LMS Project" \
                -Dsonar.sources=.
                '''
            }
        }

        stage('Build LMS') {
            steps {
                echo "LMS Build"
                sh '''
                cd webapp
                npm install
                npm run build
                '''
            }
        }

        stage('Release LMS') {
            steps {
                script {
                    def packageJson = readJSON file: 'webapp/package.json'
                    def version = packageJson.version

                    sh """
                    cd webapp
                    zip -r lms-${version}.zip dist/*

                    curl -v -u admin:nexus12345 \
                    --upload-file lms-${version}.zip \
                    http://18.119.19.122:8081/repository/lms-project/lms-${version}.zip
                    """
                }
            }
        }

        stage('Deploy LMS') {
            steps {
                script {
                    def packageJson = readJSON file: 'webapp/package.json'
                    def version = packageJson.version

                    sh """
                    curl -u admin:nexus12345 -O \
                    http://18.119.19.122:8081/repository/lms-project/lms-${version}.zip

                    sudo rm -rf /var/www/html/*
                    sudo unzip -o lms-${version}.zip -d /var/www/html/
                    """
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "Building Docker Images"

                sh '''
                docker build -t pasupunuti26/lms-fe:${BUILD_NUMBER} ./webapp
                docker build -t pasupunuti26/lms-be:${BUILD_NUMBER} ./api

                docker tag pasupunuti26/lms-fe:${BUILD_NUMBER} pasupunuti26/lms-fe:latest
                docker tag pasupunuti26/lms-be:${BUILD_NUMBER} pasupunuti26/lms-be:latest
                '''
            }
        }

        stage('Push Docker Images') {
            steps {
                echo "Pushing Docker Images"

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {

                    sh '''
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                    docker push pasupunuti26/lms-fe:${BUILD_NUMBER}
                    docker push pasupunuti26/lms-fe:latest

                    docker push pasupunuti26/lms-be:${BUILD_NUMBER}
                    docker push pasupunuti26/lms-be:latest

                    docker logout
                    '''
                }
            }
        }

    }

    post {
    success {
        echo "Pipeline Executed Successfully."
    }

    failure {
        echo "Pipeline Failed."
    }
}