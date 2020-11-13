pipeline {

    agent {
    kubernetes(k8sagent(name: 'mini+dind+k8s'))
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
        REGURL = '184881316864.dkr.ecr.us-east-1.amazonaws.com'
        ECRCRED = 'ecr:us-east-1:docker-ecr-crd'
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
                   // sh "pip install awscli ;aws --version"
                   sh """
                         apk add --no-cache \
                            python3 \
                            py3-pip \
                        && pip3 install --upgrade pip \
                        && pip3 install \
                            awscli \
                        && rm -rf /var/cache/apk/*
                      """
                        
                   //sh "apk add --update docker openrc"
                   //sh "rc-update add docker boot"
                                            // login to ECR - for now it seems that that the ECR Jenkins plugin is not performing the login as expected. I hope it will in the future.
                   // sh("eval \$(aws ecr get-login --no-include-email | sed 's|https://||')")
                      sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 184881316864.dkr.ecr.us-east-1.amazonaws.com"
                    // Push the Docker image to ECR
                      //sh "docker tag hello-world:latest 796556984717.dkr.ecr.us-east-1.amazonaws.com/hello-world:${env.BUILD_ID}"
                    docker.withRegistry(ECRURL, ECRCRED) {
                    //docker.withRegistry('ECRURL', '') {
                            dockerapp.push("latest")
                            dockerapp.push("${env.BUILD_ID}")
                            //dockerapp.image(IMAGE).push()
                    }
                    }
                }
            }
        }

    
    stage('Deploy App') {
      steps {
        script {
            container('k8s') {
          withCredentials([kubeconfigFile(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
              sh 'use $KUBECONFIG' // environment variable; not pipeline variable
              sh 'kubectl get nodes'
            }
         // kubernetesDeploy(configs: "hello-world.yml", kubeconfigId: "kubeconfig")
           kubernetesDeploy kubeconfigId: 'kubeconfig', configs: 'hello-world.yml', enableConfigSubstitution: true  // REPLACE kubeconfigId
            }
        }
      }
    }

  }
    
      
}
