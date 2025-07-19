pipeline {
    agent any

    environment {
        CI = 'true'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Solution 1: Run as current user (recommended)
                    bat '''
                        docker run --rm -u $(id -u):$(id -g) -v "%cd%:/app" -w /app node:18-alpine \
                        sh -c "npm install && npm run build && ls -la"
                    '''
                    
                    // OR Solution 2: Use a different directory inside container
                    /*
                    bat '''
                        docker run --rm -v "%cd%:/app" -w /app node:18-alpine \
                        sh -c "mkdir -p /tmp/build && cp -R /app/. /tmp/build && cd /tmp/build && npm install && npm run build && cp -R /tmp/build/. /app"
                    '''
                    */
                }
            }
        }

        stage('Unit Tests') {
            steps {
                script {
                    bat '''
                        docker run --rm -u $(id -u):$(id -g) -v "%cd%:/app" -w /app node:18-alpine \
                        sh -c "npm install && 
                               npm install --save-dev jest-junit && 
                               npm test -- --ci --reporters=default --reporters=jest-junit"
                    '''
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
                    bat '''
                        docker run --rm -u $(id -u):$(id -g) -v "%cd%:/app" -w /app -e CI=true mcr.microsoft.com/playwright:v1.39.0-jammy \
                        sh -c "npm install &&
                               npm install serve &&
                               npx playwright install --with-deps &&
                               (npx serve -s build & sleep 10) &&
                               npx playwright test --reporter=html"
                    '''
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
            when {
                expression { env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master' }
            }
            steps {
                script {
                    bat '''
                        docker run --rm -u $(id -u):$(id -g) -v "%cd%:/app" -w /app -e NETLIFY_AUTH_TOKEN -e NETLIFY_SITE_ID node:18-alpine \
                        sh -c "npm install netlify-cli &&
                               npx netlify deploy --dir=build --prod"
                    '''
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