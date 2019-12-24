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
        sh 'wget http://devtools.qiniu.com/qshell-linux-x64-v2.4.0.zip && unzip qshell-linux-x64-v2.4.0.zip'
        sh 'mv ./qshell-linux-x64-v2.4.0 /usr/local/bin/qshell'
        sh "qshell account $QINIU_ACCESSKEY $QINIU_SECRETKEY user1"
        sh 'mkdir -p upload && cp web/installer web/favicon.ico upload/'
        script {
          writeFile(
            file: 'upload.conf',
            text: """
            {
              "src_dir" : "${env.WORKSPACE}/upload/",
              "ignore_dir" : true,
              "overwrite" : true,
              "bucket" : "$QINIU_BUCKET"
            }
            """
          )
        }

        sh "qshell qupload ${env.WORKSPACE}/upload.conf"
        sh 'echo http://getcomposer.org.mirrors.china-speed.org.cn/installer > torefresh.txt'
        sh 'echo http://getcomposer.org.mirrors.china-speed.org.cn/favicon.ico > torefresh.txt'
        sh 'qshell cdnrefresh -i torefresh.txt'
        echo '部署完成'
      }
    }
  }
}
