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
                        --user node ^
                        -v "%cd%:/app" ^
                        -w /app ^
                        node:18-alpine ^
                        sh -c "npm install && npm run build && ls -la"
                    '''
                }
            }
        }

        // 👇 Add more stages (e.g., Test, Deploy) once Build succeeds.
    }

    post {
        always {
            echo "✅ Jenkins Pipeline completed"
        }
    }
}
