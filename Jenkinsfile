pipeline {
    agent any

    stages {
        // Comment
        stage('Build') {
            /*
            Big comment!
            */
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                cleanWs()
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Run tests') {
            parallel {
                stage('Static tests') {
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
                            junit 'test-results/junit.xml'
                        }
                    }
                }
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.51.1-jammy'
                            reuseNode true
                            //args '-u root:root'
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            npx playwright test --reporter=html
                        '''
                    }
                     post {
                         always {
                             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'build', reportFiles: 'index.html', reportName: 'E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                         }
                     }
                }
            }
        }
    }
}