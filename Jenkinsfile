stage('Sonar Analysis') {
    steps {
        echo 'Running SonarQube Analysis'

        sh '''
        cd webapp

        sudo docker run --rm \
          -e SONAR_HOST_URL=http://3.22.61.233:9000 \
          -e SONAR_TOKEN=sqp_15cb5716e41c9ab9bd211b67cb5d585c861c747e \
          -v $(pwd):/usr/src \
          sonarsource/sonar-scanner-cli \
          -Dsonar.projectKey=LMS-Project \
          -Dsonar.sources=.
        '''
    }
}