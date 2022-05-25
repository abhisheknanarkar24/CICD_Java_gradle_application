pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
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
                    withCredentials([<object of type com.cloudbees.jenkins.plugins.awscredentials.AmazonWebServicesCredentialsBinding>]) {

                             sh '''
                                docker build -t docker build -t 230221674655.dkr.ecr.us-east-1.amazonaws.com/java-app:${VERSION} .
                                docker push  230221674655.dkr.ecr.us-east-1.amazonaws.com/java-app:${VERSION}
                                docker rmi 230221674655.dkr.ecr.us-east-1.amazonaws.com/java-app:${VERSION}
                            '''
                    }
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
                          sh 'docker login -u $nexus_username -p $nexus_password 54.162.196.37:8083'
                          sh 'helm upgrade --install --set image.repository="54.162.196.37:8083/test-app" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                        }
               }
                    
               }
            }
        }
        

        
    }

    
}
