pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '2c9772bc-e571-401c-8b54-adb675cd79b0'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
       // CI_ENVIRONMENT_URL = 'https://playful-griffin-ec6f90.netlify.app/'  -- called on Prod instead of locally
        REACT_APP_VERSION = "1.0.$BUILD_ID"   // added this for build version ( Build_id is jenkins build number)  REACT_APP_VERSION is used in react app to show the build version on the UI
    }

    stages {

                stage('Docker') {

            steps {
                sh ''' 
                docker build -t my-playwright-app .  
                '''
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
                   // image 'mcr.microsoft.com/playwright:v1.39.0-jammy' now we are using our own docker image which has playwright and netlify-cli installed.
                    image 'my-playwright-app'
                    reuseNode true
                }
            }
            // added environment variable to avoid conflict with the default playwright-report directory for local and Prod E2E
        /*    environment {
                 PLAYWRIGHT_HTML_OUTPUT_DIR = 'playwright-report-local'
    } */

            steps {
                sh '''
                    ### npm install serve ## as serve is installed globally in the docker image, we don't need to install it again.
                    serve -s build &
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

                stage('Deploy Stage') {
                   agent {
                      docker {
                        //image 'node:18-alpine'
                        // this is fix due to netlify  Error message
                        // Command failed with ENOENT: npm run build
                        //  spawn bash ENOENT
                       // image 'node:18'  
                        image 'my-playwright-app'
                        reuseNode true
                }
                environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }
            }
            steps {
                sh '''
                    ## npm install netlify-cli
                    ## node_modules/.bin/netlify --version
                    netlify --version
                    echo "Deploying to Staging... ${NETLIFY_SITE_ID}"
                    ## node_modules/.bin/netlify status
                    netlify status
                    netlify deploy --dir=build --prod ### if we don't give --prod then it will create temp env for staging

                '''
            } 
        } 
        /*
    // adding this for apporval process before deploying to production. 
            stage('Approval') {
                steps {
                    timeout(time: 1, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                    }
                    
                }
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
        } */
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                    }
                  }

    }


}
