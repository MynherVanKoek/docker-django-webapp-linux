pipeline {
    agent any

    environment {
        DOCKER_LOGIN_SERVER = 'hoedereracr.azurecr.io'
        DOCKER_REPO_NAME = 'docker-django-webapp-linux'
        DOCKER_IMAGE = ''
        DOCKER_REGISTRY_CREDENTIAL_ID = 'hoedereracr.azurecr.io'
        AZURE_STORAGE_CREDENTIAL_ID = 'hoederermonitortest'
        AZURE_STORAGE_CONTAINER_NAME = 'jenkinslogs'

    }

    // options { timestamps () }

    stages {

        stage('Build: master')  {

            when {
                branch 'master'
            }

            environment {
                DOCKER_TAG_NAME = 'latest'
            }

            steps {

                echo "Building Docker image for $BRANCH_NAME"
                script {
                    DOCKER_IMAGE = docker.build "${DOCKER_LOGIN_SERVER}/${DOCKER_REPO_NAME}:${DOCKER_TAG_NAME}"
                }

                echo "Pushing Docker image to registry"
                script {
                    docker.withRegistry('https://' + DOCKER_LOGIN_SERVER, DOCKER_REGISTRY_CREDENTIAL_ID) {
                        DOCKER_IMAGE.push()
                    }
                }
            }
        }

        stage('Test: master')  {

            when {
                branch 'master'
            }

            environment {
                DOCKER_TAG_NAME = 'test'
            }

            steps {
                echo "TBD: Write and execute tests"
            }

        }

        stage('Deployment to staging')  {

            when {
                branch pattern: "[R|r]elease.*", comparator: "REGEXP"
            }

            steps {
                echo "TBD: Deploy Docker container"
            }

        }
        
        stage('Publish Web app') {

            environment {
                DOCKER_LOGIN_SERVER = 'hoedereracr.azurecr.io'
                DOCKER_REPO_NAME = 'docker-django-webapp-linux'
                DOCKER_REGISTRY_CREDENTIAL_ID = 'hoedereracr.azurecr.io'
                AZURE_SERVICE_PRINCIPAL_ID = 'mySp'
            }

            steps {
                
                azureWebAppPublish azureCredentialsId: AZURE_SERVICE_PRINCIPAL_ID, 
                    publishType: 'docker', 
                    resourceGroup: 'acr-test-rg', 
                    appName: 'hoederer-jenkins-app-svc', 
                    dockerImageName: "${DOCKER_LOGIN_SERVER}/${DOCKER_REPO_NAME}", 
                    dockerImageTag: 'latest', 
                    dockerRegistryEndpoint: [credentialsId: DOCKER_REGISTRY_CREDENTIAL_ID, url: 'https://' + DOCKER_LOGIN_SERVER]

            }
        }

        // stage('save log build') {

        //     steps {
        //         script {
        //             def logContent = Jenkins.getInstance()
        //                 .getItemByFullName(env.JOB_NAME)
        //                 .getBuildByNumber(
        //                     Integer.parseInt(env.BUILD_NUMBER))
        //                 .logFile.text
        //             // copy the log in the job's own workspace
        //             writeFile file: "buildlog.txt", text: logContent
        //         }
        //     }
            
        // }
    }

    post {

        always{

            // echo "${currentBuild.rawBuild.getLogInputStream()}"
            // echo "${currentBuild.rawBuild.getLogText()}"

            // sh """
            // cat ${JENKINS_HOME}/jobs/aci-helloworld/branches/master/builds/${BUILD_NUMBER}/log | grep '[0-9]:[0-9][0-9]:[0-9][0-9]' > ${WORKSPACE}/log
            // """

            sh "rm -f ${WORKSPACE}/log"

            sh """
            cat ${JENKINS_HOME}/jobs/jenkins-webapp-deployment/branches/master/builds/${BUILD_NUMBER}/log | grep -v "\\[8mha" > ${WORKSPACE}/log
            """

            sh "logger -f ${WORKSPACE}/log"

            echo "Uploading build logs ..."

            azureUpload blobProperties: [
                    cacheControl: '', 
                    contentEncoding: '', 
                    contentLanguage: '', 
                    contentType: '', 
                    detectContentType: true
                ], 
                containerName: AZURE_STORAGE_CONTAINER_NAME, 
                fileShareName: '', 
                filesPath: 'log',
                storageCredentialId: AZURE_STORAGE_CREDENTIAL_ID, 
                storageType: 'blobstorage', 
                uploadArtifactsOnlyIfSuccessful: false,
                virtualPath: "${JOB_NAME}/${BUILD_ID}/${BUILD_NUMBER}"

        }
    }    
}
