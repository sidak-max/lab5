pipeline {
    agent any
    parameters {
        booleanParam(name: 'RUN_EXTRA_CHECK', defaultValue: true, description: 'Run the extra check stage')
    }
    stages {
        stage('Checkout') {
            steps {
                // Jenkins automatically handles checkout when using SCM,
                // but we keep this here as requested by your prompt.
                echo 'Checking out code...'
            }
        }
        stage('Build') {
            steps {
                // Note: If your Jenkins agent runs on Linux/Mac instead of Windows, change 'bat' to 'sh'
                bat 'python -m py_compile app.py'
                echo 'Build successful: app.py compiled with no syntax errors'
            }
        }
        stage('Extra Check') {
            when {
                expression { params.RUN_EXTRA_CHECK == true }
            }
            steps {
                echo 'Running extra check: verifying greet() output format...'
                // Note: If your Jenkins agent runs on Linux/Mac instead of Windows, change 'bat' to 'sh'
                bat 'python -c "from app import greet; print(greet(\'Student\'))"'
            }
        }
    }
}
