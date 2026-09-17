pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                // Jenkins automatically handles checkout when using SCM,
                // but we keep this here as requested by your prompt.
                echo 'Checking out code...'
            }
        }
        stage('Generate Report') {
            steps {
                // Runs the python script. 
                // Note: If your Jenkins agent runs on Linux/Mac instead of Windows, change 'bat' to 'sh'
                bat 'python app.py'
            }
        }
        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'report.txt', fingerprint: true
            }
        }
    }
}
