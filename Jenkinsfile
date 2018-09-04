pipeline {
       agent { label "${env.PROJECT_JENKIN_AGENT}" }
       tools {
        maven 'Maven-3.5.3'
        jdk 'jdk8'       
       } 
      
        environment {
           appName = ""
           artifactId=""
           checkoutDir="source"
           PATH = "/var/lib/jenkins/data/play/play-1.5.0:$PATH"
        } 
    
        stages {            
            stage ('Initialize') {
                steps {
                sh '''
                    echo "PATH = ${PATH}"
                    cp /var/lib/jenkins/data/conf.d/ivysettings.xml ${HOME}/.ivy2/ivysettings.xml
                    cp /var/lib/jenkins/data/conf.d/settings.xml ${HOME}/.m2/settings.xml
                '''
                }
            }

            stage('Checkout GitSCM') {
                steps {                    
                    sh "if [ -d \"$checkoutDir\" ]; then rm -Rf $checkoutDir; fi"                    
                    dir ("$checkoutDir") {
                        git url: "${env.PROJECT_GIT_SOURCE}", branch: "${env.BUILD_GIT_BRANCH}", credentialsId:'b077c69a-a49a-4e08-8d4f-8ccd5758a851'
                                                
                        script {
                            artifactId=sh(
                                script: "echo \"\$(mvn -Dexec.executable='echo' -Dexec.args='\${project.artifactId}' --non-recursive exec:exec -q)\"",
                                returnStdout: true
                            ).trim()

                            appName="splus-$artifactId" 
                        }
                    
                       println "${appName}"
                    }
                    
                }
            }

            stage('Build Artifact') {
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
              steps{                  
                  dir ("$checkoutDir") {
                      script {
                       jsonYaml= sh(script: """
                       sed -e 's/<NAMESPACE>/${env.PROJECT_NAME}/g' -e 's/<BUILD_NAME>/${appName}-play-docker/g' -e 's/<SERVICE_NAME>/${artifactId}/g' /var/lib/jenkins/data/oc/play-app-docker.yaml
                       """,
                       returnStdout: true)
                       println "${jsonYaml}" 
                       //openshiftCreateResource namespace: env.PROJECT_NAME, verbose: 'true', 
                    }
                  }
                  //openshiftVerifyBuild bldCfg: "splus-${env.POM_ARTIFACT_ID}-docker", namespace: env.PROJECT_NAME
              }
            }

            stage('Deploy Docker Images') {
                 steps {
                    echo "${appName}"
                 }
            }
        }
}
