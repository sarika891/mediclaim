pipeline {
    agent {
        docker {
            image 'maven:3.8.1-adoptopenjdk-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage('SCM') {
            steps {
                git url: 'https://github.com/peddagorlavasavi/mediclaim.git'
            }
        }
        
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
                sh 'mvn clean install'
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
    }
}
