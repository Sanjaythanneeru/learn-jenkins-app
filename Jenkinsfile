pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '1fe7e51d-7f18-429f-8ec2-5d7bb6c96b9c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        stage('Docker Build') {
            agent docker
            steps {
                sh '''
                    docker build -t my-app-image .
                '''
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'my-app-image'
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
        stage('Run tests') {
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'my-app-image'
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
                            image 'my-app-image'
                            reuseNode true
                            //args '-u root:root' to run container as root
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                        '''
                        // sh npx playwright test
                    }
                }
            }
        }

        stage('Deploy stage') {
            agent {
                docker {
                    image 'my-app-image'
                    reuseNode true
                    //args '-u root:root' to run container as root
                }
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to Production site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-stage-output.json
                    echo 'Stage URL : '
                    node-jq -r '.deploy_url' deploy-stage-output.json
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
                    image 'my-app-image'
                    reuseNode true
                    //args '-u root:root' to run container as root
                }
            }
            steps {
                sh '''
                    echo 'Promoting to Prod..'
                    netlify --version
                    echo "Deploying to Production site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod --json > deploy-prod-output.json
                    echo 'Prod URL: '
                    node-jq -r '.deploy_url' deploy-prod-output.json
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