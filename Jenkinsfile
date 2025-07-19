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
                    docker.image('node:18-alpine').inside("-u root:root -v ${unixPath}:${unixPath} -w ${unixPath}") {
                        sh '''
                            node --version
                            npm --version
                            npm ci
                            npm run build
                            ls -la
                        '''
                    }
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
                            docker.image('node:18-alpine').inside("-u root:root -v ${unixPath}:${unixPath} -w ${unixPath}") {
                                sh '''
                                    npm install
                                    npm test -- --ci --reporters=default --reporters=jest-junit
                                '''
                            }
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
                            docker.image('mcr.microsoft.com/playwright:v1.39.0-jammy').inside("-u root:root -v ${unixPath}:${unixPath} -w ${unixPath}") {
                                sh '''
                                    npm ci
                                    npm install serve
                                    nohup npx serve -s build > serve.log 2>&1 &
                                    sleep 10
                                    npx playwright install --with-deps
                                    npx playwright test --reporter=html
                                '''
                            }
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
                    docker.image('node:18-alpine').inside("-u root:root -v ${unixPath}:${unixPath} -w ${unixPath}") {
                        sh '''
                            npm install netlify-cli
                            npx netlify --version
                            # Replace with actual deploy command
                            # npx netlify deploy --dir=build --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline finished."
        }
    }
}
