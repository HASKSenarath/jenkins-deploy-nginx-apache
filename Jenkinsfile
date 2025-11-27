pipeline {
    agent any

    environment {
        WEB_DIR = '/home/jenkins/app-web'
    }

    stages {
        stage('Create directory for the WEB Application') {
            steps {
                sh '''
                    # Remove directory if it exists
                    rm -rf "$WEB_DIR"
                    # (Re)create directory
                    mkdir -p "$WEB_DIR"
                '''
            }
        }

        stage('Drop the containers') {
            steps {
                echo 'Dropping the containers (if they exist)...'
                // Do not fail the build if the containers don’t exist
                sh '''
                    docker rm -f app-web-apache || true
                    docker rm -f app-web-nginx  || true
                '''
            }
        }

        // Creating the containers in parallel
        stage('Create the containers in Parallel') {
            parallel {
                stage('Create the Apache container') {
                    steps {
                        echo 'Creating the Apache container...'
                        sh '''
                            docker run -dit --name app-web-apache \
                              -p 9100:80 \
                              -v "$WEB_DIR":/usr/local/apache2/htdocs/ \
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
                              -v "$WEB_DIR":/usr/share/nginx/html \
                              nginx
                        '''
                    }
                }
            }
        }

        // Copy the application
        stage('Copy the web application to the container directory') {
            steps {
                echo 'Copying web application...'
                sh '''
                    # Ensure target is clean
                    rm -rf "$WEB_DIR"/*
                    # Copy the CONTENTS of web/ to the app-web directory
                    cp -r web/. "$WEB_DIR"/
                '''
            }
        }
    }

    post {
        success {
            echo 'The deployment in Nginx and Apache has worked'
            archiveArtifacts allowEmptyArchive: true, artifacts: 'web/**', followSymlinks: false
            cleanWs()
        }
        failure {
            echo 'An error has ocurred in the deploy'
        }
    }
}
