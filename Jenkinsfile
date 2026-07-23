pipeline {
    agent any

    stages {

        stage('Sonar Analysis') {
            steps {
                echo "LMS code analysis"
                sh '''
                cd webapp
                docker run --rm \
                -e SONAR_HOST_URL="http://3.133.130.98:9000" \
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
                    http://3.133.130.98:8081/repository/lms-project/lms-${version}.zip
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
                    http://3.133.130.98:8081/repository/lms-project/lms-${version}.zip

                    sudo rm -rf /var/www/html/*
                    sudo unzip -o lms-${version}.zip -d /var/www/html/
                    """
                }
            }
        }
    }
}    


