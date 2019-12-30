pipeline {
  agent any
  stages {
    stage('检出') {
      steps {
        checkout(
          [$class: 'GitSCM', branches: [[name: env.GIT_BUILD_REF]], 
          userRemoteConfigs: [[url: env.GIT_REPO_URL, credentialsId: env.CREDENTIALS_ID]]]
        )
      }
    }
    stage('构建') {
      steps {
        echo '构建中...'
        sh 'wget https://getcomposer.org/installer -O web/installer'
        sh 'sed -i "s|getcomposer.org\'|getcomposer.org.mirrors.china-speed.org.cn\'|g" web/installer'
        sh 'git diff'
        echo '构建完成.'
      }
    }
    stage('测试') {
      steps {
        echo '测试中...'
        sh 'php web/installer --install-dir=./ --filename=composer'
        sh './composer --version'
        echo '测试完成.'
      }
    }
    stage('部署') {
      steps {
        echo '部署中...'
        sh 'apt-get update && apt-get install -y python3-pip'
        sh 'pip3 install coscmd'
        sh "coscmd config -a $TENCENT_SECRET_ID -s $TENCENT_SECRET_KEY -b $TENCENT_BUCKET -r $TENCENT_REGION"
        sh 'coscmd upload web/installer /'
        echo '部署完成'
      }
    }
  }
}
