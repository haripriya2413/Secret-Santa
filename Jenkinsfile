pipeline {
    agent any 
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
     environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

     
    stages{
        
        stage("Git Checkout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/haripriya2413/Secret-Santa.git'
        }
        }
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test"){
            steps{
                sh "mvn test"
            }
            
        post {
            always {
          junit(testResults: 'target/surefire-reports/*.xml', allowEmptyResults : true)
        }
    }
        }
         stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Secret-Santa \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Secret-Santa '''
    
                }
            }
        }

        stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }  
        
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ ' , odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
      
        
        
       
          
         
       stage("Docker  Build & Push"){
            steps{
               script{
                     withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                        sh "docker build -t secreatsanta ."
                        sh "docker tag secreatsanta priya247/secreatsanta:${env.BUILD_NUMBER} "
                        sh "docker push priya247/secreatsanta:${env.BUILD_NUMBER} "
                    
                    }
                }
            }
       }
        stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "Secret-Santa"
            GIT_USER_NAME = "haripriya2413"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "haripriyagh@gmail.com"
                    git config user.name "haripriya2413"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" manifests/deploymentservice.yml
                    git add manifests/deploymentservice.yml
                    git add -A
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    git rebase main
                '''
            }
        }
    }

             stage("Deploy To Kuberates Cluster"){
        steps {
        sh "kubectl apply -f manifests/deploymentservice.yml"
     }
    }
        
       

    }
}


