pipeline {
  agent any
   environment {
    dockerImage = ''
     uuid_hash = UUID.randomUUID().toString()
  }
  
  tools {nodejs "node"}

  stages {
    stage('Cloning Git') {
      steps {
        git 'https://github.com/HarveyD/node-jenkins-app-example.git'
      }
    }
        
    stage('Install dependencies') {
      steps {
        sh 'npm install'
      }
    }
     
    stage('Test') {
      steps {
         sh 'npm test'
      }
    } 
     stage('Building image') {
      steps{
        script{
          dockerImage = docker.build("nodejs:${uuid_hash}")
          echo "dockerImage: ${dockerImage}"
        }
      }
    }
    stage("Push to ECR") {
        steps {
        //cleanup current user docker credentials
        sh 'rm  ~/.dockercfg || true'
        sh 'rm ~/.docker/config.json || true'
        script{
         //configure registry
            docker.withRegistry('https://747701862635.dkr.ecr.ap-southeast-2.amazonaws.com', 'ecr:ap-southeast-2:aws') {
                //push image
                dockerImage.push()
              }
            }
        }
    }
    
  }
}

