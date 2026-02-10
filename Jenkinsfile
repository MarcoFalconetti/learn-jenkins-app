pipeline {
    agent any

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
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
                // Salvo la build per gli stage paralleli
                stash includes: 'build/**', name: 'build-folder'
            }
        }

        stage('Stage Test') {
            parallel {

                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        unstash 'build-folder'
                        sh '''
                            npm ci
                            npm test
                        '''
                        stash includes: 'jest-junit.xml', name: 'unit-results', allowEmpty: true
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        unstash 'build-folder'
                        sh '''
                            npm ci
                            npx serve -s build &
                            sleep 10
                            npx playwright test --reporter=junit --output=test-results
                        '''
                        stash includes: 'test-results/*.xml', name: 'e2e-results', allowEmpty: true
                    }
                }

            }
        }

    }

    post {
        always {
            // recupero i risultati delle due pipeline parallele
            unstash 'unit-results'
            unstash 'e2e-results'

            // pubblicazione report JUnit
            junit '**/*.xml'
        }
    }
}