pipeline {
  agent any
  stages {
    stage('Setup') {
      steps {
        sh 'script/setup'
      }
    }
    stage('Test') {
      steps {
        sh 'script/test'
      }
    }
    stage('Build artifact') {
      steps {
        sh 'script/cibuild'
      }
    }
    stage('Deploy') {
      steps {
        sh 'docker run -t --volume ${PWD}:/data --volume ${HOME}/Desktop/vagrantbox.pem:/root/Desktop/vagrantbox.pem --workdir /data/deploy --rm webstores/ansible-docker:3.1.0 ansible-playbook -i hosts.ini deploy.yml -l production'
      }
    }
  }
  post {
    always {
      junit "**/build/xml/*.xml"
    }
  }
  environment {
    TEST = 'test'
  }
}
