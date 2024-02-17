pipeline {
  agent any
  environment {
      PATH = "/var/lib/jenkins/.nvm/versions/node/v19.9.0/bin:${env.PATH}"
      DOCKER_REGISTRY = "docker.io"
      DOCKERHUB_CREDENTIALS = credentials('DOCKERHUB_CREDENTIALS')
      VERSION = "${env.BUILD_ID}"
  }


tools {
    nodejs "Nodejs"
}

  stages {

 stage('Install Dependencies') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Build Project') {
      steps {
        // Build the Angular project
        sh 'npm run build'
      }
    }


    stage('Docker Build and Push') {
      steps {
          withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_CREDENTIALS', passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW', usernameVariable: 'DOCKERHUB_CREDENTIALS_USR')]) {
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh 'docker build -t cristianchira/food-delivery-app-fe:${VERSION} .'
          sh 'docker push cristianchira/food-delivery-app-fe:${VERSION}'
          }
      }
    }


     stage('Cleanup Workspace') {
      steps {
        deleteDir()
      }
    }

     stage('Update Image Tag in GitOps') {
      steps {
        // checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-ssh', url: 'git@github.com/cristianchira/deployment-folder.git']]])
      checkout([
          $class: 'GitSCM', 
          branches: [[name: '*/master']],
          extensions: [],
          userRemoteConfigs: [[
              credentialsId: 'git-ssh', 
              url: 'git@github.com:cristianchira/deployment-folder.git'
         ]]
      ])

        script {
          // Set the new image tag with the Jenkins build number
       sh '''
          sed -i "s/image:.*/image: cristianchira\\/food-delivery-app-fe:${VERSION}/" aws/angular-manifest.yml
        '''

          sh 'git checkout master'
          // Set Git user email and name for this repository
          sh 'git config user.email "cristianchira@gmail.com"'
          sh 'git config user.name "Cristian CHIRA"'
          sh 'git add .'
          sh 'git commit -m "Update image tag"'
        sshagent(['git-ssh'])
            {
                  sh('git push')
            }
        }
      }
    }
  }

}


