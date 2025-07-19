pipeline {
    agent any

    environment {
        CI = 'true'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    bat '''
                        docker run --rm ^
                        -v "%cd%:/app" ^
                        -w /app ^
                        node:18-alpine ^
                        sh -c "npm install && npm run build && ls -la"
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
