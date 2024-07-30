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
                            [artifactId: backend,
                            classifier: '',
                            file: "backend-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )
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
    