pipeline {
    agent {
        label 'AGENT-1'
    }
    options { 
        // Timeout counter starts BEFORE agent is allocated
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment {
        def appVersion = ''
        def nexusUrl = 'nexus.puneeth.cloud:8081'
    }
    parameters {
        string(name: 'appVersion', defaultValue: '1.0.0', description: 'What is the application version?')
        booleanParam(name: 'deploy', defaultValue: false, description: 'whether to deploy or not')
    }
    stages {
        stage('Read the Version') {
            steps {
                script {
                  def packageJson = readJSON file: 'package.json'
                  appVersion = packageJson.version
                  echo "app version : $appVersion"
                }                
            }
        }
        stage('Install Dependencies') {
            steps {
                sh """
                 npm install
                 ls -ltr
                 echo app version : $appVersion 
                """
            }
        }
        stage ('Build') {
            steps {
                sh """
                zip -q -r backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
                ls -ltr 
                """
            }
        }
        stage ('Sonar scan') {
            environment {
                scannerHome = tool 'sonar-6.0'   // reffering Sonar scanner CLI
            }
            steps {
                script {
                    withSonarQubeEnv('sonar-6.0') {  // reffering Sonar Server
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 30, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        stage ('Nexus Artifact Upload') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: 'backend',
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: "backend",
                            classifier: '',
                            file: "backend-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )
                }
            }
        }
        stage ('Deploy') {
            when {
                expression{
                  params.deploy
               }
            }
            steps {
                script {
                    def params = [
                string(name: 'appVersion', value: "${appVersion}")
                ]
                    build job: 'backend-deploy', parameters: params, wait: false
                }                
            }
        }
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success{
            echo 'I will when pipeline is success'
        }
        failure{
            echo 'I will when pipeline is failure'
        }
    }
}
    

