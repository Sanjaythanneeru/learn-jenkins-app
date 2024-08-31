pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '1fe7e51d-7f18-429f-8ec2-5d7bb6c96b9c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        /*
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
                npm ci  #npm ci is used instead npm install as git won't include node_modules folder
                npm run build
                ls -la
                ls
                '''
            }
        }
        */
        stage('Run tests') {
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo 'This is test stage'
                            #test -f build/index.html
                            npm test
                        '''
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            //args '-u root:root' to run container as root
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test
                        '''
                    }
                }
            }
        }

        stage('Deploy stage') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    //args '-u root:root' to run container as root
                }
            }
            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to Production site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build > deploy-stage-output.json
                    echo "Stage URL: ${node_modules/.bin/node-jq -r '.deploy_url' deploy-stage-output.json}"
                '''
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    input 'Promote stage to Prod?'
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    //args '-u root:root' to run container as root
                }
            }
            steps {
                sh '''
                    echo 'Promoting to Prod..'
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to Production site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod > deploy-prod-output.json
                    echo "Prod URL: ${node_modules/.bin/node-jq -r '.deploy_url' deploy-prod-output.json}"
                '''
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}