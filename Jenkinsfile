pipeline {
agent any
stages {
stage('Checkout') {
steps {
git branch: 'main', url: 'https://github.com/<student-username>/<repo-name>.git'
}
}
stage('Parallel Checks') {
parallel {
stage('Frontend Check') {
steps {
3
bat 'python frontend_check.py'
}
}
stage('Backend Check') {
steps {
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
