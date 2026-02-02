pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '30ca17d5-2d3b-4c0c-b336-10846c0b976b'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.2.$BUILD_ID"
    }

    stages {

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }

            steps {
                sh '''
                    aws --version
                '''
            }
        }

        // This is a comment
        stage('Build') {
            agent {
                docker {
                    image 'node:18-bullseye'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Small change to trigger the build..."
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls
                '''
            }
        }

        stage ('Tests') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_PLACEHOLDER'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to Staging site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'STAGING Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy Production') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://genuine-flan-c34ccf.netlify.app'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to Production site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PROD Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }   
}
