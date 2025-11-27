pipeline {
    agent any

    stages {
        stage('Drop the containers') {
            steps {
                echo 'Dropping the containers (if they exist)...'
                sh '''
                    docker rm -f app-web-apache 2>/dev/null || true
                    docker rm -f app-web-nginx  2>/dev/null || true
                '''
            }
        }

        // Create Apache and Nginx containers in parallel
        stage('Create the containers in Parallel') {
            parallel {
                stage('Create the Apache container') {
                    steps {
                        echo 'Creating the Apache container...'
                        sh '''
                            docker run -dit --name app-web-apache \
                              -p 9100:80 \
                              httpd
                        '''
                    }
                }
                stage('Create the Nginx container') {
                    steps {
                        echo 'Creating the Nginx container...'
                        sh '''
                            docker run -dit --name app-web-nginx \
                              -p 9200:80 \
                              nginx
                        '''
                    }
                }
            }
        }

        // Copy the application into the running containers
        stage('Copy the web application to the containers') {
            steps {
                echo 'Copying web application into Apache and Nginx containers...'
                sh '''
                    # Copy the CONTENTS of the "web" folder in this repo
                    # into each container's document root.
                    docker cp web/. app-web-apache:/usr/local/apache2/htdocs/
                    docker cp web/. app-web-nginx:/usr/share/nginx/html/
                '''
            }
        }
    }

    post {
        success {
            echo 'The deployment in Nginx and Apache has worked'
            // Archive the source files from the repository
            archiveArtifacts allowEmptyArchive: true, artifacts: 'web/**', followSymlinks: false
            // Clean only the Jenkins workspace (containers keep their files)
            cleanWs()
        }
        failure {
            echo 'An error has occurred in the deploy'
        }
    }
}
