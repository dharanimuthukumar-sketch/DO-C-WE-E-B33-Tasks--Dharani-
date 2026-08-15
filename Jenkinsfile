pipeline {
    agent any
    
    stages {
        stage('Run Script') {
            steps {
                sh 'chmod +x myscript.sh'
                sh './myscript.sh'
            }
        }
    }
    
    post {
        always {
            mail (
                to: 'dharani.muthukumar@gmail.com',
                subject: "Jenkins Build #${env.BUILD_NUMBER} - ${currentBuild.currentResult}: Job ${env.JOB_NAME}",
                body: """Jenkins Build Details:
                         Job Name: ${env.JOB_NAME}
                         Build Number: ${env.BUILD_NUMBER}
                         Status: ${currentBuild.currentResult}
                         Check Console Output here: ${env.BUILD_URL}console""",
                attachLog: true
            )
        }
    }
}    

