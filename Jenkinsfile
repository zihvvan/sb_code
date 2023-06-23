pipeline {
  agent any
  // any, none, label, node, docker, dockerfile, kubernetes
  tools {
    maven 'my_maven'
  }
  
  environment {
    GITNAME = 'zihvvan'
    GITEMAIL = 'kimji217@naver.com'
    GITWEBADD = 'https://github.com/zihvvan/sb_code.git'
    GITSSHADD = 'git@github.com:zihvvan/sb_code.git'
    GITDEPADD = 'git@github.com:zihvvan/sb_code.git'
    GITCREDENTIAL = 'git_cre'
    // github credential 생성시의 ID
    DOCKERHUB = '211.183.3.10:5000/myhttpd'
    DOCKERHUBCREDENTIAL = 'docker_cre' 
    // docker credential 생성시의 ID
  }

  stages {
    stage('Checkout Github') {
      steps {
        slackSend (channel: '#jenkins_zihvvan', color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [],
                  userRemoteConfigs: [[credentialsId: GITCREDENTIAL, url: GITWEBADD]]])
      }  
      post {
        failure {
          echo 'Repository clone failure'
        }
        success {
          echo 'Repository clone success'
        }
      }
    }
    stage('Maven Build') {
      steps {
        sh 'mvn clean install'
        // maven 플러그인이 미리 설치 되어있어야 함.
      }
      post {
        failure {
          echo 'maven build failure'
        }
        success {
          echo 'maven build success'
        }
      }
    }
    stage('Docker image Build') {
      steps {
        sh "docker build -t ${DOCKERHUB}:${currentBuild.number} ."
        sh "docker build -t ${DOCKERHUB}:latest ."
        // oolralra/sbimage:4 이런식으로 빌드가 될것이다.
        // currentBuild.number 젠킨스에서 제공하는 빌드넘버변수.
      }
      post {
        failure {
          echo 'docker image build failure'
        }
        success {
          echo 'docker image build success'
        }
      }
    }
    stage('docker image push') {
      steps {  
            sh "docker push ${DOCKERHUB}:${currentBuild.number}"
            sh "docker push ${DOCKERHUB}:latest"
       

      }
      post {
        failure {
          echo 'docker image push failure'
          sh "docker image rm -f ${DOCKERHUB}:${currentBuild.number}"
          sh "docker image rm -f ${DOCKERHUB}:latest"
        }
        success {
          sh "docker image rm -f ${DOCKERHUB}:${currentBuild.number}"
          sh "docker image rm -f ${DOCKERHUB}:latest"  
          echo 'docker image push success'
        }
      }
    }
    stage('k8s manifest file update') {
      steps {
        git credentialsId: GITCREDENTIAL,
            url: GITDEPADD,
            branch: 'main'
        
        // 이미지 태그 변경 후 메인 브랜치에 푸시
        sh "git config --global user.email ${GITEMAIL}"
        sh "git config --global user.name ${GITNAME}"
        sh "sed -i 's@${DOCKERHUB}:.*@${DOCKERHUB}:${currentBuild.number}@g' deploy/deployment.yml"
        sh "git add ."
        sh "git commit -m 'fix:${DOCKERHUB} ${currentBuild.number} image versioning'"
        sh "git branch -M main"
        sh "git remote remove origin"
        sh "git remote add origin ${GITDEPADD}"
        sh "git push -u origin main"

      }
      post {
        failure {
          echo 'k8s manifest file update failure'
        }
        success {
          echo 'k8s manifest file update success'  
        }
      }
    }

  }
}
