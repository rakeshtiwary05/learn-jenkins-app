pipeline {
    agent any

    environment {
        CI = 'true'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    def containerId = bat(script: "docker create node:18-alpine sh -c \"npm ci && npm run build && ls -la\"", returnStdout: true).trim()
                    bat "docker cp . ${containerId}:/app"
                    bat "docker start -a ${containerId}"
                    bat "docker rm ${containerId}"
                }
            }
        }

        stage('Unit Tests') {
            steps {
                script {
                    def containerId = bat(script: "docker create node:18-alpine sh -c \"npm install && npm test -- --ci --reporters=default --reporters=jest-junit\"", returnStdout: true).trim()
                    bat "docker cp . ${containerId}:/app"
                    bat "docker start -a ${containerId}"
                    bat "docker rm ${containerId}"
                }
            }
            post {
                always {
                    junit 'jest-junit.xml'
                }
            }
        }

        stage('E2E') {
            steps {
                script {
                    def containerId = bat(script: "docker create mcr.microsoft.com/playwright:v1.39.0-jammy sh -c \"npm ci && npm install serve && nohup npx serve -s build > serve.log 2>&1 & sleep 10 && npx playwright install --with-deps && npx playwright test --reporter=html\"", returnStdout: true).trim()
                    bat "docker cp . ${containerId}:/app"
                    bat "docker start -a ${containerId}"
                    bat "docker rm ${containerId}"
                }
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright Test Report'
                    ])
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def containerId = bat(script: "docker create node:18-alpine sh -c \"npm install netlify-cli && npx netlify --version\"", returnStdout: true).trim()
                    bat "docker cp . ${containerId}:/app"
                    bat "docker start -a ${containerId}"
                    bat "docker rm ${containerId}"
                }
            }
        }
    }

    post {
        always {
            echo "✅ Jenkins Pipeline completed"
        }
    }
}
