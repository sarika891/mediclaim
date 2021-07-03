pipeline {
    agent any
    stages {
        
        stage('slack notify') {
            steps {
                slackSend color: '#439FE0', message: "started ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|JobLink>)"
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
        stage('OWASP:Dependency check') {
            steps {
                      sh 'mvn dependency-check:check'
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
                sh 'mvn clean install deploy'
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
        stage ('Deploy via ansible') {
            steps {
            ansiblePlaybook become: true, vaultCredentialsId: 'ansiblevault', disableHostKeyChecking: true, extras: '-e app_name=mediclaim', installation: 'ansible', inventory: 'deploy/inventories/dev/hosts', playbook: 'deploy/deploy.yml'
            }
        }
         stage('Run Application') {
            steps {
                sh 'java -jar /opt/mediclaim/mediclaim.jar &'
            }
        }
    }
  post { 
       success {
           slackSend color: "good", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} was successful (<${env.BUILD_URL}|JobLink>)"
       }
       // triggered when red sign
       failure {
           slackSend color: "danger", message: "Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} was failed (<${env.BUILD_URL}|JobLink>)"
       }
      always { 
            cleanWs()
        }
}
}
