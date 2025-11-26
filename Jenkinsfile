pipeline {
    agent any
    
    stages {
        stage('Create  directory for the WEB Application')        
       {
            steps {
                //First, drop the directory if exists
                sh 'rm -rf /home/jenkins/app-web'
                //Create the directory
                sh 'mkdir /home/jenkins/app-web'               
            }
        }
        stage('Drop the containers'){   
            
            steps {
            echo 'droping the container...'
            sh 'docker rm -f app-web-apache'
            sh 'docker rm -f app-web-nginx'
            }
        }
        //Creating the containers in Parallel
        stage('Create the containers in Parallel') {
            parallel{
                stage('Create the Apache container')
                {           
                    steps {
                    echo 'Creating the Apache Container...'
                    sh 'docker run -dit --name app-web-apache -p 9100:80  -v /home/jenkins/app-web:/usr/local/apache2/htdocs/ httpd'
                    }
                }
                stage('Create the Nginx container') {
                   steps {
                   echo 'Creating the Apache container...'
                   sh 'docker run -dit --name app-web-nginx -p 9200:80  -v /home/jenkins/app-web:/usr/share/nginx/html nginx'         
                   }
                }       
            }   
        }
        //Copy the application   
        stage ('Copy the web application to the container directory') {
            steps {
                echo 'Copying web application...'
                // Copy the CONTENTS of web/ to the app-web directory
                sh 'cp -r web/. /home/jenkins/app-web'
            }
        }
    }

    post {              
        success {
        // One or more steps need to be included within each condition's block.
          echo 'The deployment in Nginx and Apache has worked'
          archiveArtifacts allowEmptyArchive: true, artifacts: 'web/*', followSymlinks: false
          cleanWs()         
       }
       failure {
        // One or more steps need to be included within each condition's block.
        echo 'An error has ocurred in the deploy'       
       }
    }
}
