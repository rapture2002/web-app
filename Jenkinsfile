COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
  ]
pipeline 
{
    agent any
    tools {
        maven "maven3.9.6"
    }
    
    stages{
        stage("git clone") {
            steps {
                git branch: 'main', url: 'https://github.com/rapture2002/web-app.git'
            }
        }

        stage("build with maven") {
            steps {
                sh "mvn clean"
            }
        }
        stage("testing with maven") {
            steps {
                sh "mvn test"
            }
        }
        stage("packaging with maven") {
            steps {
                sh "mvn package"
            }
        }
        stage('sonar analysis') {
            environment {
                ScannerHone = tool 'sonar5.0'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh '${ScannerHome}/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar5.0/bin/sonar-scanner -Dsonar.projectKey=eddie'
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
          }
        }
        stage ("upload to Nexus") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/first-pipeline/target/web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com.mt', nexusUrl: '172.16.91.133:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-Release/', version: '3.0.6-Release'
            }
        }
        stage("Deploy to UAT") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'Tomcat-c', path: '', url: 'http://172.16.91.132:8080')], 
                contextPath: null, 
                war: 'target/*.war'
            }
        }
        stage('email'){
            steps {
        emailext body: '''Build is over

        JOMACS 
        43
        }
        */
        7212483''', recipientProviders: [developers(), requestor()], subject: 'Build', to: '1stpvtech@gmail.com'
            } 
        }
    }
    post {
        success {
            slackSend channel: 'jenkins-notifications', color: 'good', message: "Build successful: ${currentBuild.fullDisplayName}"
        }
        failure {
            slackSend channel: 'jenkins-notifications', color: 'danger', message: "Build failed: ${currentBuild.fullDisplayName}"
        }
    } 
}