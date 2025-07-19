pipeline {
    agent any

    environment {
        CI = 'true'
        UNIX_WORKSPACE = '/c/ProgramData/Jenkins/.jenkins/workspace/learn-Jenkins-app' // update path if needed
    }

    stages {
        stage('Build') {
            steps {
                script {
                    docker.image('node:18-alpine').inside("-u root:root -v ${env.UNIX_WORKSPACE}:${env.UNIX_WORKSPACE} -w ${env.UNIX_WORKSPACE}") {
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
                            docker.image('node:18-alpine').inside("-u root:root -v ${env.UNIX_WORKSPACE}:${env.UNIX_WORKSPACE} -w ${env.UNIX_WORKSPACE}") {
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
                            docker.image('mcr.microsoft.com/playwright:v1.39.0-jammy').inside("-u root:root -v ${env.UNIX_WORKSPACE}:${env.UNIX_WORKSPACE} -w ${env.UNIX_WORKSPACE}") {
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
                    docker.image('node:18-alpine').inside("-u root:root -v ${env.UNIX_WORKSPACE}:${env.UNIX_WORKSPACE} -w ${env.UNIX_WORKSPACE}") {
                        sh '''
                            npm install netlify-cli
                            npx netlify --version
                            # Replace with actual Netlify deploy command:
                            # npx netlify deploy --dir=build --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed"
        }
    }
}
