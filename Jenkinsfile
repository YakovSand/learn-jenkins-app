pipeline {
    agent any

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
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
                            npm ci
                            # Run tests
                            npm test
                            echo "Tests completed successfully"
                        '''
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
                }
            }
        }
       
        
        // stage('Run Tests') {
        //     parallel {
        //         stage('Test') {
        //                 agent {
        //                     docker {
        //                         image 'node:18-alpine'
        //                         reuseNode true
        //                     }
        //                 }
        //                 steps {
        //                     sh '''
        //                         pwd
        //                         ls -l build
        //                         echo "Running tests"
        //                         # Check build/index.html exists

        //                         if [ ! -f build/index.html ]; then
        //                             echo "build/index.html not found, exiting"
        //                             exit 1
        //                         fi
        //                         echo "
        //                         build/index.html found, proceeding with tests"
        //                         # Install dependencies
        //                         npm ci
        //                         # Run tests
        //                         npm test
        //                         echo "Tests completed successfully"
        //                     '''
        //                 }
        //         }
        //         stage('E2E Test') {
        //             agent {
        //                 docker {
        //                     image 'mcr.microsoft.com/playwright:v1.54.0-noble'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh '''
        //                     npm install serve
        //                     node_modules/.bin/serve -s build & 
        //                     sleep 10
        //                     npx playwright install
        //                     npx playwright test --reporter=html
        //                 '''
        //             }
        //         }
        //     }
        // }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
