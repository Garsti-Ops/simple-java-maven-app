pipeline {
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            when {
              branch '*'
            }
            steps {
                sh 'mvn -B -D skipTests clean package'
            }
        }
        stage('Test') {
            when {
              branch '*'
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
    }
}
