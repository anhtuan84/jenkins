pipeline {
       agent { label "${env.PROJECT_JENKIN_AGENT}" }
       tools {
        maven 'Maven-3.5.3'
        jdk 'jdk8'
       } 
      
        environment {
           appName = ""
        } 
    
        stages {
        
            stage('Checkout GitSCM') {
                steps {
                    sh 'if [ -d "source_dir" ]; then rm -Rf source_dir; fi'
                    
                    dir ('source_dir-dir') {
                        git url: "${env.PROJECT_GIT_SOURCE}", branch: "${env.BUILD_GIT_BRANCH}", credentialsId:'b077c69a-a49a-4e08-8d4f-8ccd5758a851'
                        
                        sh 'mvn clean package'
                        script {
                            appName=sh(
                                script: "echo \"splus-\$(mvn -Dexec.executable='echo' -Dexec.args='\${project.artifactId}' --non-recursive exec:exec -q)\"",
                                returnStdout: true
                            ).trim()
                        }
                    
                       println "${appName}"
                    }
                    
                }
            }
            stage('Build') {
                 steps {
                    echo "${appName}"
                 }
            }
        }
  
   

}
