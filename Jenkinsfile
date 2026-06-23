pipeline {
agent any

environment {
    NEXUS_CRED = credentials('nexus')
}

stages {

    stage('Build') {
        steps {
            echo 'Building LMS Frontend...'
            sh '''
                cd webapp
                npm install
                npm run build
            '''
        }
    }

    stage('SonarQube Scan') {
        steps {
            echo 'Running SonarQube Analysis...'
            sh '''
                cd webapp

                sudo docker run --rm \
                -e SONAR_HOST_URL="http://3.22.61.233:9000" \
                -e SONAR_LOGIN="sonar-token" \
                -v $(pwd):/usr/src \
                sonarsource/sonar-scanner-cli \
                -Dsonar.projectKey=LMS-Project-
            '''
        }
    }

    stage('Release to Nexus') {
        steps {
            echo 'Uploading Artifact to Nexus...'

            sh '''
                cd webapp

                rm -f dist-${BUILD_NUMBER}.zip

                zip -r dist-${BUILD_NUMBER}.zip dist

                curl -v \
                -u ${NEXUS_CRED_USR}:${NEXUS_CRED_PSW} \
                --upload-file dist-${BUILD_NUMBER}.zip \
                http://3.22.61.233:8081/repository/lms/dist-${BUILD_NUMBER}.zip
            '''
        }
    }
}

post {
    success {
        echo 'Pipeline Completed Successfully'
    }

    failure {
        echo 'Pipeline Failed'
    }
}
}
