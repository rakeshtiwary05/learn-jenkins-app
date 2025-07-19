pipeline {
    agent any

    environment {
        CI = 'true'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    def winPath = pwd()
                    def unixPath = winPath.replace('C:\\', '/c/').replace('\\', '/')
                    def dockerCmd = """docker run --rm -v ${unixPath}:${unixPath} -w ${unixPath} node:18-alpine sh -c "node --version && npm --version && npm ci && npm run build || true && ls -la || true" """ 
                    bat "${dockerCmd}"
                }
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    steps {
                        script {
                            def winPath = pwd()
                            def unixPath = winPath.replace('C:\\', '/c/').replace('\\', '/')
                            def dockerCmd = """docker run --rm -v ${unixPath}:${unixPath} -w ${unixPath} node:18-alpine sh -c "npm install && npm test -- --ci --reporters=default --reporters=jest-junit" """ 
                            bat "${dockerCmd}"
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
                            def winPath = pwd()
                            def unixPath = winPath.replace('C:\\', '/c/').replace('\\', '/')
                            def dockerCmd = """docker run --rm -v ${unixPath}:${unixPath} -w ${unixPath} mcr.microsoft.com/playwright:v1.39.0-jammy sh -c "npm ci && npm install serve && nohup npx serve -s build > serve.log 2>&1 & sleep 10 && npx playwright install --with-deps && npx playwright test --reporter=html" """ 
                            bat "${dockerCmd}"
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
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def winPath = pwd()
                    def unixPath = winPath.replace('C:\\', '/c/').replace('\\', '/')
                    def dockerCmd = """docker run --rm -v ${unixPath}:${unixPath} -w ${unixPath} node:18-alpine sh -c "npm install netlify-cli && npx netlify --version" """ 
                    bat "${dockerCmd}"
                    // Optional: Uncomment below to deploy with Netlify
                    // def deployCmd = """docker run --rm -v ${unixPath}:${unixPath} -w ${unixPath} node:18-alpine sh -c "npx netlify deploy --dir=build --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID" """ 
                    // bat "${deployCmd}"
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
