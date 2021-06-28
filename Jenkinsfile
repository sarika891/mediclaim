pipeline {
    agent any
    stages {
        
        stage('SonarQube:Code Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                      sh 'mvn clean package sonar:sonar'
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Publish Test Coverage Report') {
           steps {
              step([$class: 'JacocoPublisher', 
                   execPattern: '**/build/jacoco/*.exec',
                   classPattern: '**/build/classes',
                   sourcePattern: 'src/main/java',
                   exclusionPattern: 'src/test*'
                   ])
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
        stage('Unit Test') { 
            steps {
                sh 'mvn test' 
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml' 
                }
            }
        }
        stage ('Deploy') {
            steps {
            sh 'ssh ubuntu@18.157.177.45 mkdir -p /tmp/test'
            sh 'scp -r target ubuntu@18.157.177.45:/tmp/test'
            }
        }
    }
}
