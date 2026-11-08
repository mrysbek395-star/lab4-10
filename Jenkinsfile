pipeline {
    agent any

    environment {
        IMAGE = 'lab4-10'
    }

    stages {

        stage('Secret scan') {
            steps {
                sh '''
                    trivy fs \
                    --scanners secret \
                    --skip-dirs .git \
                    --exit-code 1 .
                '''
            }
        }

        stage('Build image') {
            steps {
                sh '''
                    docker build \
                    -t ${IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Image gate') {
            steps {
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    sh '''
                        trivy image \
                        --pkg-types library \
                        --severity HIGH,CRITICAL \
                        --ignore-unfixed \
                        --exit-code 1 \
                        ${IMAGE}:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('OS report') {
            steps {
                sh '''
                    trivy image \
                    --pkg-types os \
                    --severity CRITICAL \
                    --ignore-unfixed \
                    ${IMAGE}:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        always {
            sh '''
                docker rmi -f ${IMAGE}:${BUILD_NUMBER} || true
            '''
        }
    }
}
