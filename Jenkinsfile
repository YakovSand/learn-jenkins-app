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
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    pwd
                    ls -l build
                    echo "Running tests"
                    # Check build/index.html exists

                    if [ ! -f build/index.html ]; then
                        echo "build/index.html not found, exiting"
                        exit 1
                    fi
                    echo "
                    build/index.html found, proceeding with tests"
                    # Install dependencies
                    npm ci
                    # Run tests
                    npm test
                    echo "Tests completed successfully"
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
