pipeline {
    agent any
    environment {
      DOCKER_ACCOUNT =
      DOCKER_REPO =
      DOCKERREPO =
    }
    stages {
        // Gradle Build
        stage('Gradle Build') {
            steps {
                withGradle() {
                    sh './gradlew build'
                }

                archiveArtifacts(artifacts: 'build/libs/*.jar', onlyIfSuccessful: true)
            }
        } 
        // Merge Before Build (( Check if merge Acceptable or Not ))
        /* stage('Merge Before Build'){
            steps{
                script{
                   checkout([$class: 'GitSCM', branches: [[name: "$env.BRANCH_NAME"]], extensions: [[$class: 'PreBuildMerge', options: [mergeRemote: 'origin', mergeStrategy: 'RECURSIVE_THEIRS', mergeTarget: 'feature/testtarget']]], userRemoteConfigs: [[credentialsId: 'upwork', url: ']]])
                }
            }
        }
        */
        // Build and push App image
        stage('Build && push app') {
            when { expression { BRANCH_NAME ==~ /(master|stage|dev)/ } }
            steps {
                sh "sudo docker build -f ./services/app/Dockerfile -t $DOCKER_ACCOUNT/$DOCKERREPO:app-$env.BRANCH_NAME . "
                sh "sudo docker push $DOCKER_ACCOUNT/$DOCKERREPO:app-$env.BRANCH_NAME"            
            }
        }
        // Build and push Cover image
        stage('Build and push Cover') {
            when { expression { BRANCH_NAME ==~ /(master|stage|dev)/ } }
            steps {
                sh "sudo docker build -f ./services/cover/Dockerfile -t $DOCKER_ACCOUNT/$DOCKERREPO:cover-$env.BRANCH_NAME . "
                sh "sudo docker push $DOCKER_ACCOUNT/$DOCKERREPO:cover-$env.BRANCH_NAME"            
            }               

        }
        // Build and push Home Image
        stage('Build and push home') {
            when { expression { BRANCH_NAME ==~ /(master|stage|dev)/ } }
            steps {
                sh "sudo docker build -f ./services/home/Dockerfile -t $DOCKER_ACCOUNT/$DOCKERREPO:home-$env.BRANCH_NAME . "
                sh "sudo docker push $DOCKER_ACCOUNT/$DOCKERREPO:home-$env.BRANCH_NAME"            
            }
        }
        // Build and push UAA Image
        stage('Build and push uaa') {
            when { expression { BRANCH_NAME ==~ /(master|stage|dev)/ } }
            steps {
                sh "sudo docker build -f ./uaa/Dockerfile -t $DOCKER_ACCOUNT/$DOCKERREPO:uaa-$env.BRANCH_NAME . "
                sh "sudo docker push $DOCKER_ACCOUNT/$DOCKERREPO:uaa-$env.BRANCH_NAME"            
            }     
        }
        // Build and push Gateway Imae
        stage('Build and push gateway') {
            when { expression { BRANCH_NAME ==~ /(master|stage|dev)/ } }
            steps {
                sh "sudo docker build -f ./gateway/Dockerfile -t $DOCKER_ACCOUNT/$DOCKERREPO:gw-$env.BRANCH_NAME . "
                sh "sudo docker push $DOCKER_ACCOUNT/$DOCKERREPO:gw-$env.BRANCH_NAME"            
            }    
        }    
        // Deploy to Deployment Server     
        stage('Deploy to Deployment server') {
            when { expression { BRANCH_NAME ==~ /(master|stage|dev)/ } }
            steps{
                sh "ssh $REMOTE_USER@$DEPLOYMENT_SERVER 'cd /home/$env.BRANCH_NAME && docker-compose pull && docker-compose up -d'"
            }            
        }
        // Merge Feature branches if Build successfull 
        
         stage('merge feature to develop') {
          when { branch "feature/*" }
          steps {
               script{
                  git branch: 'feature/testtarget', credentialsId: 'cred1', url: ''
                  withCredentials([usernamePassword(credentialsId: 'cred1', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]){    
                    sh ''' git config --local credential.helper "!f() { echo username=\\$GIT_USERNAME; echo password=\\$GIT_PASSWORD; }; f"
                         '''
                    sh "git checkout $env.BRANCH_NAME"     
                    sh "git checkout -B feature/testtarget"
                    sh "git merge $env.BRANCH_NAME"
                    sh "git push origin feature/testtarget"
                  
                  }     
               }

          }
          post{
            failure{
                sh "echo can't merge"
            }
          }
        } 
        
        
        // Update Gira with build info     
        stage('Jira') {
           steps {
               jiraSendBuildInfo(site: '' branch: 'develop')
           }
        }

        stage('Remove Unused docker image ') {
            steps {
                sh 'sudo docker image prune -a --filter "until=24h"'
            }
         }
    }
}
