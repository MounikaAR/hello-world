pipeline {

    agent {
    kubernetes(k8sagent(name: 'mini+dind'))
  }
options
    {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
      environment 
    {
        VERSION = 'latest'
        PROJECT = 'hello-world'
        IMAGE = 'hello-world:latest'
        ECRURL = 'http://184881316864.dkr.ecr.us-east-1.amazonaws.com'
        ECRCRED = 'ecr:eu-central-1:docker_ecr'
    }
  stages {

    stage('Checkout Source') {
      steps {
        
        script {
          container('docker') {
             sh 'docker ps'
            //git url:'https://github.com/vamsijakkula/hellowhale.git', branch:'master'
          }
        }
        
        //git branch: 'master', credentialsId: 'gitbubnewcrd', url: 'https://github.com/MounikaAR/hello-world.git'
      }
    }
      
      stage('Build preparations')
        {
            steps
            {
                script 
                {
                    container('alpine/git') {
                    // calculate GIT lastest commit short-hash
                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    shortCommitHash = gitCommitHash.take(7)
                    // calculate a sample version tag
                    VERSION = shortCommitHash
                    // set the build display name
                    currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                    IMAGE = "$PROJECT:$VERSION"
                    }
                }
            }
        }
    
      stage("Build image") {
            steps {
                script {
                    container('docker') {
                        dockerapp = docker.build("${IMAGE}")
                    }
                }
            }
        }
    
      stage("Push image") {
            steps {
                script {
                    container('docker') {
                        
                         // login to ECR - for now it seems that that the ECR Jenkins plugin is not performing the login as expected. I hope it will in the future.
                    sh("eval \$(aws ecr get-login --no-include-email | sed 's|https://||')")
                    // Push the Docker image to ECR
                    docker.withRegistry(ECRURL, ECRCRED)
                    {
                    //docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            dockerapp.push("latest")
                           // dockerapp.push("${env.BUILD_ID}")
                            dockerapp.image(IMAGE).push()
                    }
                    }
                }
            }
        }

    
    stage('Deploy App') {
      steps {
        script {
            container('kubectl') {
          kubernetesDeploy(configs: "hellowhale.yml", kubeconfigId: "kubeconfig")
            }
        }
      }
    }

  }
    
      
}
