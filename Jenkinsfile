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
    stages {
        stage('Read the Version') {
            steps {
                scripts{
                def packageJson = readJSON file: 'package.json'
                def appVersion = packageJson.version
                echo "$appVersion"
                }                
            }
        }
        stage('Install Dependencies') {
            steps {
                sh """
                 npm install
                 ls -ltr
                 echo $appVersion 
                """
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
    