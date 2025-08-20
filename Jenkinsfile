pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '2d2430f4-321d-4042-a7e7-5124eb541192'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token') // Assuming you have set up a Jenkins credential with this ID
    }

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
        stage('Run Tests') {
            parallel {
                stage('Unit Tests') {
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
                            # npm ci
                            # Run tests
                            npm test
                            echo "Tests completed successfully"
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E Test') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.54.0-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build & 
                            sleep 10
                            npx playwright install
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Deploying the application"
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to Production.Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --prod --dir=build
                    echo "Deployment completed successfully"
                '''
            }
        }
    }
}
