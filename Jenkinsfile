pipeline {
    agent any

    stages {
      stage('Build') {
        steps {
          script {
            dockerImage = docker.build("ehsan309/distance-converter:${env.BUILD_ID}")
        }
    }
}
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'ls -l index.html'
            }
        }

        stage('Deploy') {
            steps {
                script {
                   
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "server1", 
                                transfers: [sshTransfer(
                                    execCommand: """
                                        docker pull ehsan309/distance-converter:${env.BUILD_ID}
                                        docker stop distance-converter-container || true
                                        docker rm distance-converter-container || true
                                        docker run -d --name distance-converter-container -p 80:80 ehsan309/distance-converter:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )

                  
                    boolean isDeploymentSuccessful = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http://16.171.231.92:80', returnStdout: true).trim() == '200'

                    if (!isDeploymentSuccessful) {
                       
                        def previousSuccessfulTag = readFile('previous_successful_tag.txt').trim()
                        sshPublisher(
                            publishers: [
                                sshPublisherDesc(
                                    configName: "server1",
                                    transfers: [sshTransfer(
                                        execCommand: """
                                            docker pull ehsan309/distance-converter:${previousSuccessfulTag}
                                            docker stop distance-converter-container || true
                                            docker rm distance-converter-container || true
                                            docker run -d --name distance-converter-container -p 80:80 ehsan309/distance-converter:${previousSuccessfulTag}
                                        """
                                    )]
                                )
                            ]
                        )
                    } else {
                       
                        writeFile file: 'previous_successful_tag.txt', text: "${env.BUILD_ID}"
                    }
                }
            }
        }
    }

    post {
        failure {
            mail(
                to: 'sp20-bcs-040@cuiatk.edu.pk',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: """Something is wrong with the build ${env.BUILD_URL}
                Rolling back to the previous version

                Regards,
                Jenkins
                
                """
            )
        }
    }
}
