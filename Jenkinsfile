pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '2c9772bc-e571-401c-8b54-adb675cd79b0'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
       // CI_ENVIRONMENT_URL = 'https://playful-griffin-ec6f90.netlify.app/'  -- called on Prod instead of locally
        REACT_APP_VERSION = "1.0.${BUILD_ID}"  // for versioning along with BUILD_ID of jenkins  
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

        stage('AWS CLI') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args '--entrypoint=""'
                    reuseNode true
                }

            environment {
                AWS_S3_BUCKET = 'learn-jenkins-29072026'
            }
            steps {

                withCredentials([usernamePassword(credentialsId: 'aws-token-s3', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version
                    ## aws configure list
                    ## aws s3 ls  
                    ## echo "Hello S3 from Jenkins" > index.html
                    ## aws s3 cp index.html s3://$AWS_S3_BUCKET/index.html
                    aws sync build s3://$AWS_S3_BUCKET  ## --delete this will delete the files from S3 which are not present in build folder. this is good pratice to discard old unused files.
                '''
                  }
                  }
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

                stage('Deploy Stage') {
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
            } 
        } 
    // adding this for apporval process before deploying to production. 
         /*   stage('Approval') {
                steps {
                    timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Kindly approve for deployment ...', ok: 'Yes, approving the deployment'
                    }
                    
                }
            } */ //commenting this stage for now as no need to approve for now. ( 129-136 )
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
