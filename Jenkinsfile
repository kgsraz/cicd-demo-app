pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/kgsraz/cicd-demo-app.git', description: 'HTTPS URL of the repository')
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'Branch to build')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run unit tests')
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${params.GIT_BRANCH}"]],
                    userRemoteConfigs: [[url: params.REPO_URL]]
                ])
                script {
                    def branchName = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    def commitSha  = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Branch: ${branchName}"
                    echo "Commit SHA: ${commitSha}"
                    env.GIT_COMMIT_SHA = commitSha
                }
            }
        }

        stage('Validate') {
            steps {
                script {
                    def required = ['README.md', 'requirements.txt', 'src', 'tests']
                    required.each { item ->
                        if (!fileExists(item)) {
                            error("Обязательный файл/каталог отсутствует: ${item}")
                        }
                    }
                    echo "Структура проекта валидна"
                }
            }
        }

        stage('Test') {
            when {
                expression { return params.RUN_TESTS }
            }
            steps {
                sh 'python3 -m unittest discover -s tests -v'
            }
        }

        stage('Build') {
            steps {
                script {
                    writeFile file: 'build-info.txt', text: """Build Number: ${env.BUILD_NUMBER}
Commit SHA: ${env.GIT_COMMIT_SHA}
"""
                    sh '''
                        rm -f app.tar.gz
                        tar -czf /tmp/app.tar.gz --exclude=.git .
                        mv /tmp/app.tar.gz .
                    '''
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'app.tar.gz', fingerprint: true
            }
        }
    }

    post {
        success { echo "Сборка #${env.BUILD_NUMBER} успешно завершена" }
        failure { echo "Сборка #${env.BUILD_NUMBER} завершилась с ошибкой" }
        always  { echo "Pipeline завершён. Статус: ${currentBuild.currentResult}" }
    }
}
