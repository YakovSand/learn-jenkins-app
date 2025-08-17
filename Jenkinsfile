pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    echo "Installing dependencies and building the project"
                    # Install dependencies and build the project
                    npm ci
                    # Build the project
                    npm run build
                    echo "Build completed successfully"
                    ls -la
                '''
            }
        }
    }
}
