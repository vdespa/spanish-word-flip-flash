pipeline {
    agent any

    options {
        ansiColor('xterm')
    }

    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:22-alpine'
                }
            }
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('test') {
            parallel {
                stage('unit tests') {
                    agent {
                        docker {
                            image 'node:22-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        // Install dependencies in the test container, then run unit tests with Vitest
                        sh 'npm ci'
                        sh 'npm exec -- vitest run --reporter=verbose'
                    }
                }
                stage('integration tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh 'npm ci'
                        sh 'npm exec -- playwright test'
                    }
                }
            }
        }

        stage('deploy') {
            agent {
                docker {
                    image 'alpine'
                }
            }
            steps {
                // Mock deployment which does nothing
                echo 'Mock deployment was successful!'
            }
        }
        stage('e2e') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                    reuseNode true
                }
            }
            environment {
                E2E_BASE_URL = 'https://spanish-cards.netlify.app/'
            }
            steps {
                sh 'npm ci'
                sh 'npm exec -- playwright test'
            }
            post {
                always {
                    // Collect Playwright test results and screenshots for debugging
                    publishHTML(allowMissing: true, alwaysLinkToLastBuild: true, icon:'',keepAll: false, reportDir: 'reports-e2e/html/', reportFiles: 'index.html', reportName: 'Playwright Test HTML Report', reportTitles:'', useWrapperFileDirectly:true)
                    junit stdioRetention: 'ALL', testResults: 'reports-e2e/junit/*.xml'
                }
            }
        }
    }
}
