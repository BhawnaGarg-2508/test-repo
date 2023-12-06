pipeline {

    stages {
        stage('Checkout') {
            steps {
                // checkout scm: [
                //      $class: 'GitSCM',
                //      branches: [[name: 'main']],
                //      extensions: [],
                //      userRemoteConfigs: [[url:'https://github.com/Amit2508/House-price-prediction-.git']]
                //  ]
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    sh "echo 'Building'"
                }
            }
        }


        stage('Deploy') {
            steps {
                script {
                    try {
                        echo 'Deploying the application'

                        // Log into Docker
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"

                        // Build Docker image
                        sh "docker build -t ${DOCKER_IMAGE_NAME} ."

                        // Run Docker container with port exposure
                        sh "docker run -d -p 8000:3000 --name ${DOCKER_CONTAINER_NAME} ${DOCKER_IMAGE_NAME}"

                        // Wait for the web app to start
                        sleep time: 30, unit: 'SECONDS'

                        // Print Docker container logs for debugging
                        sh "docker logs ${DOCKER_CONTAINER_NAME}"
                    } catch (Exception deployException) {
                        currentBuild.result = 'FAILURE'
                        throw deployException
                    }
                }
            }
        }
    }

    post {
        failure {
            script {
                echo 'Before email notification'

                // Stop and remove the Docker container
                sh 'docker stop ${DOCKER_CONTAINER_NAME} || true'
                sh 'docker rm ${DOCKER_CONTAINER_NAME} || true'

                // Send email notification with web app URL and failure details
                emailext subject: "Web App Build and Test Results - ${currentBuild.result}",
                    body: """
                    See Jenkins console output for details.

                    Web App URL: http://your-jenkins-server:8000

                    Failure Details:
                    - Build: ${currentBuild.result == 'FAILURE' ? 'Failed' : 'Successful'}
                    - Unit Test: ${currentBuild.result == 'FAILURE' ? 'Failed' : 'Successful'}
                    - Deployment: ${currentBuild.result == 'FAILURE' ? 'Failed' : 'Successful'}
                    """,
                    recipientProviders: [
                        [$class: 'CulpritsRecipientProvider'],
                        [$class: 'DevelopersRecipientProvider'],
                        [$class: 'RequesterRecipientProvider']
                    ],
                    replyTo: '$DEFAULT_REPLYTO',
                    to: '$DEFAULT_RECIPIENTS'

                echo 'After email notification'
            }
        }
    }
}