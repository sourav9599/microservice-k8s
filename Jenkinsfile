pipeline {
    agent {
        label 'master'
    }
    environment {
        
        DOCKERHUB_CREDENTIALS=credentials('docker-cred')
        dockerImage = ''
    }

    stages {

        stage('Sonarqube Analysis') {
            steps {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                        cd $microservice
                        mvn clean verify sonar:sonar -Dsonar.projectKey=$microservice
                        '''
                    }
            }
        }
        
        stage ('Exec Maven') {
            steps {
               sh '''
               cd $microservice
               mvn clean package
               '''
            }
        }

        stage('Building image') {
            steps {
                    sh '''
                    cd $microservice 
                    docker build -t us.gcr.io/balmy-particle-334205/$microservice-app:latest .
                    '''
                
            }
        }
        stage('Upload Image') {
            steps {
                    sh 'gcloud docker -- push us.gcr.io/balmy-particle-334205/$microservice-app:latest'
                }
            }
        stage('Deploy to GKS') {
            steps {
                    sh '''
                    gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project balmy-particle-334205
                    cd $microservice
                    kubectl apply -f kubernetes/$microservice.yml
                    '''
                }
            }
    }
       

        // stage('docker stop container') {
        //     steps {
        //         sh 'docker ps -f name=user-service -q | xargs --no-run-if-empty docker container stop'
        //         sh 'docker container ls -a -fname=user-service -q | xargs -r docker container rm'
        //     }
        // }
        // stage('Docker Run') {
        //     steps {
        //         script {
        //             dockerImage.run('-p 4001:4001 --rm --name user-service')
        //         }
        //     }
        // }
    
    post {
        always {
            cleanWs()
            sh 'docker logout'
        }
    }
}
