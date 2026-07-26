pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '2c9772bc-e571-401c-8b54-adb675cd79b0'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
       // CI_ENVIRONMENT_URL = 'https://playful-griffin-ec6f90.netlify.app/'  -- called on Prod instead of locally
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
        

        stage('Test'){
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
                    #test -f build/index.html
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
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            // added environment variable to avoid conflict with the default playwright-report directory for local and Prod E2E
        /*    environment {
                 PLAYWRIGHT_HTML_OUTPUT_DIR = 'playwright-report-local'
    } */

            steps {
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright local', reportTitles: '', useWrapperFileDirectly: true])
                    }
                  }
        }


            }

        }

   /*                     stage('Deploy Stage') {
                   agent {
                      docker {
                        //image 'node:18-alpine'
                        // this is fix due to netlify  Error message
                        // Command failed with ENOENT: npm run build
                        //  spawn bash ENOENT
                        image 'node:18'
                        reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to Staging... ${NETLIFY_SITE_ID}"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build   ### if we don't give --prod then it will create temp env for staging

                '''
            } */
        }
                stage('Deploy Prod') {
                   agent {
                      docker {
                        //image 'node:18-alpine'
                        // this is fix due to netlify  Error message
                        // Command failed with ENOENT: npm run build
                        //  spawn bash ENOENT
                        image 'node:18'
                        reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production... ${NETLIFY_SITE_ID}"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod ##--no-build

                '''
            }
        }
        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://playful-griffin-ec6f90.netlify.app'
                // added this environment variable to avoid conflict with the default playwright-report directory for local and Prod E2E
               // PLAYWRIGHT_HTML_OUTPUT_DIR = 'playwright-report-prod'
            }

            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                    }
                  }
        }

    }


}
