pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID= 'b9ba31dd-3f09-48ca-9a9f-8a5bc2a3537a'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')

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

/*                 stage('E2E'){
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
                } */
            }
        }


        stage('Deploy staging') {
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
                    node_modules/.bin/netlify --version
                    echo "Deploying to stging. Project ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

        stage('Deploy production') {
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
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Project ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        /* stage('Prod E2E'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.52.0-noble       '
                    reuseNode true
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'https://strong-choux-7c5c72.netlify.app'
            }

            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }

            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        } */
    }
}
