pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = 'f8c0d589-6470-4e56-bc0d-25b9ff44d30f'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
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
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }        
        stage('Run Tests'){
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }            
                    steps{
                        sh '''
                            echo "Test Stage"
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
                stage('Local E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }            
                    steps{
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright --version
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright: Local Report', reportTitles: ''])
                        }
                    }                    
                }
            }
        }
        stage('Deploy Staging') {
            environment{
                CI_ENVIRONMENT_URL="TEST_STAGING_URL"
            }
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    netlify --version
                    netlify status
                    echo "Deploying to Staging!"
                    netlify deploy --dir=build --json > deploy-output.json                
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    npx playwright --version
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright: Staging E2E Report', reportTitles: ''])
                }
            }
        }        
        stage('Deploy Prod') {
            environment{
                CI_ENVIRONMENT_URL='https://curious-froyo-ff7012.netlify.app'
            }
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    netlify --version
                    netlify status
                    echo "Deploying to netlify!"
                    netlify deploy --dir=build --prod                
                    echo "Starting E2E test!"
                    npx playwright --version
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright: Prod Report', reportTitles: ''])
                }
            }
        }
    }
}
