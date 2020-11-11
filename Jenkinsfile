pipeline {

    agent {
    kubernetes(k8sagent(name: 'mini+dind'))
  }

  stages {

    stage('Checkout Source') {
      steps {
        
        script {
          container('docker') {
            docker ps
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
                    myapp = docker.build("sivisoft/hellowhale:${env.BUILD_ID}")
                    }
                }
            }
        }
    
      stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

    
    stage('Deploy App') {
      steps {
        script {
          kubernetesDeploy(configs: "hellowhale.yml", kubeconfigId: "kubeconfig")
        }
      }
    }

  }

}
