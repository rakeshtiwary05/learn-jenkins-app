pipeline {
    agent any

    environment {
        CI = 'true'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    def containerId = bat(script: 'docker create -w /app node:18-alpine sh -c "npm install && npm run build && ls -la"', returnStdout: true).trim()
                    bat "docker cp . ${containerId}:/app"
                    bat "docker start -a ${containerId}"
                    bat "docker rm ${containerId}"
                }
            }
        }

        stage('Unit Tests') {
            steps {
                script {
                    def containerId = bat(script: '''
                        docker create -w /app node:18-alpine sh -c "
                        npm install &&
                        npm install --save-dev jest-junit &&
                        npm test -- --ci --reporters=default --reporters=jest-junit"
                    ''', returnStdout: true).trim()

                    bat "docker cp . ${containerId}:/app"
                    bat "docker start -a ${containerId}"

                    // Copy test report from container to host
                    bat "docker cp ${containerId}:/app/jest-junit.xml jest-junit.xml"
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
                    def containerId = bat(script: '''
                        docker create -w /app mcr.microsoft.com/playwright:v1.39.0-jammy sh -c "
                        npm install &&
                        npm install serve &&
                        nohup npx serve -s build > serve.log 2>&1 & sleep 10 &&
                        npx playwright install --with-deps &&
                        npx playwright test --reporter=html"
                    ''', returnStdout: true).trim()

                    bat "docker cp . ${containerId}:/app"
                    bat "docker start -a ${containerId}"

                    // Copy Playwright report from container to host
                    bat "docker cp ${containerId}:/app/playwright-report ./playwright-report"
                    bat "docker rm ${containerId}"
                }
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
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
                    def containerId = bat(script: '''
                        docker create -w /app node:18-alpine sh -c "
                        npm install netlify-cli &&
                        npx netlify --version"
                    ''', returnStdout: true).trim()

                    bat "docker cp . ${containerId}:/app"
                    bat "docker start -a ${containerId}"

                    // Uncomment below when ready to deploy
                    // bat "docker exec ${containerId} npx netlify deploy --dir=build --auth=${NETLIFY_AUTH_TOKEN} --site=${NETLIFY_SITE_ID}"

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
