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
        stage('Parallel Checks') {
            parallel {
                stage('Frontend Check') {
                    steps {
                        // Note: If your Jenkins agent runs on Linux/Mac instead of Windows, change 'bat' to 'sh'
                        bat 'python frontend_check.py'
                    }
                }
                stage('Backend Check') {
                    steps {
                        // Note: If your Jenkins agent runs on Linux/Mac instead of Windows, change 'bat' to 'sh'
                        bat 'python backend_check.py'
                    }
                }
            }
        }
        stage('Summary') {
            steps {
                echo 'Both frontend and backend checks are complete.'
            }
        }
    }
}

}
}
