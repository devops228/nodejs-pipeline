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
    
    stage("Scan image"){
     steps{
	sh '''
        docker run -d --name db arminc/clair-db
        sleep 15 # wait for db to come up
        docker run -p 6060:6060 --link db:postgres -d --name clair arminc/clair-local-scan
        sleep 1
        DOCKER_GATEWAY=$(docker network inspect bridge --format "{{range .IPAM.Config}}{{.Gateway}}{{end}}")
        wget -qO clair-scanner https://github.com/arminc/clair-scanner/releases/download/v8/clair-scanner_linux_amd64 && chmod +x clair-scanner
        ./clair-scanner --ip="$DOCKER_GATEWAY" ${uuid_hash}:latest || exit 0
      '''
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

