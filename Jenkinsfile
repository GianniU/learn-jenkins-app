pipeline {
    agent any

    environment{
        NETFLY_SITE_ID= 'b9ba31dd-3f09-48ca-9a9f-8a5bc2a3537a'
    }

     stages {
        stage('build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root:root'
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
            parallel{
                stage('test'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                            args '-u root:root'
                        }
                    }
                    steps{
                        sh '''
                            echo "Test stage"
                            #test -f build/index.html
                            npm test
                        '''
                    }

                    post{
                        always{
                            junit 'test-unit-results/junit.xml'
                        }
                    }
                }

                stage('E2E'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.52.0-noble       '
                            reuseNode true
                            args '-u root:root'
                        }
                    }
                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root:root'
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    netlify --version
                    echo "Deploying to production. Project ID: $NETFLY_SITE_ID"
                '''
            }
        }
    }
}
