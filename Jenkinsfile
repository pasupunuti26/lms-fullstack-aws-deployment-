pipeline {
    agent any

    stages {

        stage('Sonar Analysis') {
            steps {
                echo "LMS code analysis"
                sh '''
                cd webapp
                docker run --rm \
                -e SONAR_HOST_URL="http://18.118.208.219:9000" \
                -e SONAR_TOKEN="sqp_15cb5716e41c9ab9bd211b67cb5d585c861c747e" \
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
                    http://18.118.208.219:8081/repository/lms-project/lms-${version}.zip
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
                    http://18.118.208.219:8081/repository/lms-project/lms-${version}.zip

                    sudo rm -rf /var/www/html/*
                    sudo unzip -o lms-${version}.zip -d /var/www/html/
                    """
                }
            }
        }
    }
}    

