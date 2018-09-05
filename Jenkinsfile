pipeline {
       agent { label "${env.PROJECT_JENKIN_AGENT}" }

       options {
          ansiColor('xterm')
       }

       tools {
        maven 'Maven-3.5.3'
        jdk 'jdk8'       
       } 
      
        environment {
           appName = ""
           artifactId = ""
           buildName = "BUILDNAME"
           checkoutDir = "source"
           gitSource = "${env.PROJECT_GIT_SOURCE}"
           gitBranch = "${env.BUILD_GIT_BRANCH}"           
           gitCredentialsId = "${env.GIT_CREDENTIALS_ID}"
           projectNamespace = "${env.PROJECT_NAME}"
           projectPrefix = "${env.PROJECT_PREFIX}"
           PATH = "/var/lib/jenkins/data/play/play-1.5.0:$PATH"           
        } 
    
        stages {            
            stage ('Initialize') {
                steps {
                sh '''                    
                    cp /var/lib/jenkins/data/conf.d/ivysettings.xml ${HOME}/.ivy2/ivysettings.xml
                    cp /var/lib/jenkins/data/conf.d/settings.xml ${HOME}/.m2/settings.xml
                '''
                }
            }

            stage('Checkout GitSCM') {
                when { expression { gitCredentialsId != null } }
                steps {                    
                    sh "if [ -d \"$checkoutDir\" ]; then rm -Rf $checkoutDir; fi"
                    dir ("$checkoutDir") {
                        git url: "$gitSource", branch: "$gitBranch", credentialsId:"$gitCredentialsId"
                                                
                        script {
                            artifactId=sh(
                                script: "echo \"\$(mvn -Dexec.executable='echo' -Dexec.args='\${project.artifactId}' --non-recursive exec:exec -q)\"",
                                returnStdout: true
                            ).trim()

                            appName="$projectPrefix-$artifactId" 
                            buildName="${appName}-play-docker"
                        }
                    
                       println "${appName}"
                    }
                    
                }
            }

            stage('Build Artifact') {
               when { expression { env.USE_CACHED == null } }
               steps {
                   dir ("$checkoutDir") {
                       sh 'mvn clean package'

                       dir("docker") {
                           sh "cp ../target/*.zip ./${appName}.zip"
                       }
                   }     
                }
            }

            stage('Build Docker Images') {
              when { expression { appName != "" } }  
              steps{ 
                  dir ("$checkoutDir") {
                      dir("docker") {
                        script {                           
                            sh """
                            sed -e 's/<NAMESPACE>/${projectNamespace}/g' -e 's/<BUILD_NAME>/${buildName}/g' -e 's/<SERVICE_NAME>/${artifactId}/g' /var/lib/jenkins/data/oc/play-app-docker.yaml  > play-app-docker.yaml
                            """
                            def bc = readYaml file: 'play-app-docker.yaml'
                            openshift.withCluster() {
                                openshift.withProject("${projectNamespace}") {
                                   def objs = openshift.selector('bc',"${buildName}")
                                   if (!objs.exists()) {
                                     objs = openshift.create(bc)
                                   }
                                   objs.startBuild("--from-file=.", "--wait")
                                }
                            }
                        }
                      }
                  }
              }
            }

            stage('Deploy Docker Images') {
                 steps {
                    echo "Deploy ${appName} ..."
                    script {
                       openshift.withCluster() {
                            openshift.withProject("${projectNamespace}") {
                                openshift.raw("rollout latest dc/${appName}")
                                def latestDeploymentVersion = openshift.selector('dc',"${appName}").object().status.latestVersion
                                def rc = openshift.selector('rc', "${appName}-${latestDeploymentVersion}")
                                rc.untilEach(1){
                                    def rcMap = it.object()
                                    return (rcMap.status.replicas.equals(rcMap.status.readyReplicas))
                                }
                            }
                        }
                    }
                 }
            }

            stage('Clean Up') {
                steps {
                    script {
                        openshift.withCluster() {
                            openshift.withProject("${projectNamespace}") {
                                def objs = openshift.selector('bc',"${buildName}")
                                if (objs.exists()) {
                                     objs.delete()
                                }                                
                            }
                        }
                    }
                }
            }
        }
}
