pipeline {

    environment {
        IMAGE_NAME = "staticwebserver"
        IMAGE_TAG = "ajc-2.0"
        PRODUCTION = "sofiane-ajc-staticwebserver-prod-env"
        CONTAINER_NAME = "staticwebserver_container"
        EC2_PRODUCTION_HOST = "34.235.138.52"
    }

    agent none

    stages{

       stage ('Build Image'){
           agent any
           steps {
               script{
                   sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
               }
           }
       }

       stage ('Run test container') {
           agent any
           steps {
               script{
                   sh '''
                       docker stop $CONTAINER_NAME || true
                       docker rm $CONTAINER_NAME || true
                       docker run --name $CONTAINER_NAME -d -p 5000:80 $IMAGE_NAME:$IMAGE_TAG
                       sleep 5
                   '''
               }
           }
       }
        
        stage ('clean env') {
           agent any
           steps {
               script{
                   sh '''
                       docker stop $CONTAINER_NAME || true
                       docker rm $CONTAINER_NAME || true
                   '''
               }
           }
       }

        
        
         stage('Deploy app on EC2-cloud Production') {
            agent any
            when{
                expression{ GIT_BRANCH == 'origin/prod'}
            }
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: "ec2_produc_private_key", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        script{ 

                            timeout(time: 15, unit: "MINUTES") {
                                input message: 'Do you want to approve the deploy in production?', ok: 'Yes'
                            }

                            sh'''
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker run --name $CONTAINER_NAME -d -p 5000:80 $IMAGE_NAME:$IMAGE_TAG
                            '''
                            }
                        }
                    }
                }
             script{
                 if (currentBuild.result == "SUCCESS") {
                     slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
                 }
                 else {
                     slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")'
                 }
             }
         }
        
    
}
