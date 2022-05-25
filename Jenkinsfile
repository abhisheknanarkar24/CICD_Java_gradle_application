pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
        AWS_ACCOUNT_ID="230221674655"
        AWS_DEFAULT_REGION="us-east-1" 
        IMAGE_REPO_NAME="java-app"
        IMAGE_TAG="${VERSION}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
    stages{
        //stage("sonar quality check"){
        //    agent {
        //        docker {
        //            image 'gradle-docker'
        //        }
        //    }
        //    steps{
        //        script{
        //            withSonarQubeEnv(credentialsId: 'sonarqube') {
        //                    sh 'cd /opt/gradle/gradle-7.5-rc-1/'
        //                    sh 'chmod +x gradlew'
        //                    sh './gradlew sonarqube --warning-mode all'
        //            }
        //           timeout(time: 1, unit: 'HOURS') {
        //              def qg = waitForQualityGate()
        //              if (qg.status != 'OK') {
        //                   error "Pipeline aborted due to quality gate failure: ${qg.status}"
        //              }
        //            }
        //       }  
        //    }
        //}
        stage("docker build & docker push"){
            steps{
                script{
                

                             sh '''
                                aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
                                docker build -t 230221674655.dkr.ecr.us-east-1.amazonaws.com/java-app:${VERSION} .
                                docker push  230221674655.dkr.ecr.us-east-1.amazonaws.com/java-app:${VERSION}
                                docker rmi 230221674655.dkr.ecr.us-east-1.amazonaws.com/java-app:${VERSION}
                            '''
                    
                }
            }
        }
        stage('indentifying misconfigs using datree in helm charts'){
            steps{
                script{
                  dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=06736911-56d1-416f-aaa2-4c872f7f821f']) {
                              sh '''
                                 helm datree test myapp/
                            '''
                        }
                    }
                }
            }
        }
        stage("pushing the helm charts to nexus"){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'nexus_password', usernameVariable: 'nexus_username')]) {
                              dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 curl -u $nexus_username:$nexus_password http://54.162.196.37:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                          }
                    }
                }
            }
        }

        

        stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'nexus_password', usernameVariable: 'nexus_username')]) {
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="230221674655.dkr.ecr.us-east-1.amazonaws.com/java-app" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                        }
               }
                    
               }
            }
        }
        

        
    }

    
}
