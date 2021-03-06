pipeline {
  agent any
  environment {
    FRONTEND_GIT = 'https://github.com/ThoArrow/tutorial-jenkins-frontend.git'
    FRONTEND_BRANCH = 'master'
    FRONTEND_IMAGE = 'thoarrown/tutorial-jenkins-frontend'
    FRONTEND_SERVER = '35.240.207.69'
    FRONTEND_SERVER_DIR = '/home/app'
  }
  stages {
    stage('Build JS') {
      agent {
        docker {
          image 'node:latest'
          args '-v tutorial_jenkins_frontend_modules:$WORKSPACE/node_modules'
        }
      }
      steps {
        git(url: FRONTEND_GIT, branch: FRONTEND_BRANCH)
        sh 'npm i'
        sh 'npm run build'
        stash(name: 'frontend', includes: 'build/*/**')
      }
    }
    stage('Build Image') {
      steps {
        unstash 'frontend'
        script {
          docker.withRegistry('', 'dockerhub') {
            def image = docker.build(FRONTEND_IMAGE)
            image.push(BUILD_ID)
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        script {
          sh 'docker ps'
          sh 'date && cal'
          withCredentials([sshUserPrivateKey(
            credentialsId: 'ssh',
            keyFileVariable: 'identityFile',
            passphraseVariable: '',
            usernameVariable: 'thoarrow'
          )]) {
            def remote = [:]
            remote.name = 'server'
            remote.host = FRONTEND_SERVER
            remote.user = thoarrow
            remote.identityFile = identityFile
            remote.allowAnyHosts = true

            sshCommand remote: remote, command: "sudo su && date && cal"
          }
        }
      }
    }
  }
}