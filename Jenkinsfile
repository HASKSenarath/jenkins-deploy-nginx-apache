pipeline {
    agent any

    // Directory that both Jenkins and Docker share (inside /var/jenkins_home)
    environment {
        APP_DIR = "${env.WORKSPACE}/app-web"
    }

    stages {
        stage('Create directory for the WEB Application') {
            steps {
                sh '''
                    # Clean and recreate the app directory
                    rm -rf "$APP_DIR"
                    mkdir -p "$APP_DIR"
                '''
            }
        }

        stage('Drop the containers') {
            steps {
                echo 'Dropping the containers (if they exist)...'
                // Do not fail the build if containers are not present
                sh '''
                    docker rm -f app-web-apache || true
                    docker rm -f app-web-nginx  || true
                '''
            }
        }

        // Creating the containers in Parallel
        stage('Create the containers in Parallel') {
            parallel {
                stage('Create the Apache container') {
                    steps {
                        echo 'Creating the Apache container...'
                        sh '''
                            docker run -dit --name app-web-apache \
                              -p 9100:80 \
                              -v "$APP_DIR":/usr/local/apache2/htdocs/ \
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
                              -v "$APP_DIR":/usr/share/nginx/html \
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
                    # Make sure target dir is clean
                    rm -rf "$APP_DIR"/*
                    # Copy the CONTENTS of web/ into the shared directory
                    cp -r web/. "$APP_DIR"/
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
