import hudson.model.Result
import hudson.model.*;
import jenkins.model.CauseOfInterruption
node {
}

def skipbuild=0
def win_stop=0

def abortPreviousBuilds() {
  def currentJobName = env.JOB_NAME
  def currentBuildNumber = env.BUILD_NUMBER.toInteger()
  def jobs = Jenkins.instance.getItemByFullName(currentJobName)
  def builds = jobs.getBuilds()

  for (build in builds) {
    if (!build.isBuilding()) {
      continue;
    }

    if (currentBuildNumber == build.getNumber().toInteger()) {
      continue;
    }

    build.doKill()    //doTerm(),doKill(),doTerm()
  }
}
//  abort previous build
abortPreviousBuilds()
def abort_previous(){
  def buildNumber = env.BUILD_NUMBER as int
  if (buildNumber > 1) milestone(buildNumber - 1)
  milestone(buildNumber)
}
def pre_test(){
    sh'echo ${WK}'
    sh'hostname'
    sh '''
    cd ${WK}
    echo ${WK}
    git reset --hard HEAD~10 >/dev/null
    '''
    script {
      echo env.CHANGE_TARGET
      if (env.CHANGE_TARGET == 'master') {
        sh '''
        cd ${WK}
        git checkout master
        '''
      }
      // else if(env.CHANGE_TARGET == '2.0'){
      //   sh '''
      //   cd ${WK}
      //   git checkout 2.0
      //   '''
      // }
      else {
        sh '''
        cd ${WK}
        git checkout develop
        '''
      }
    }
    sh '''
    cd ${WK}
    git pull >/dev/null
    git fetch origin +refs/pull/${CHANGE_ID}/merge
    git checkout -qf FETCH_HEAD
    git clean -dfx
    '''
    return 1
}

pipeline {
  agent {label 'master'}
  options { skipDefaultCheckout() } 
  environment{
      WK = '/var/lib/jenkins/workspace/server-sdk-java'
  }
  stages {
    stage('pre_build'){
      options { skipDefaultCheckout() } 
      when {
        changeRequest()
      }
      steps {
        script{
          echo 'pre_build'
          abort_previous()
          abortPreviousBuilds()
        }
      }
    }
    stage('Maven Build') {
      when {
        changeRequest()
      }
      steps {
        pre_test()
        script {
          sh '''
          cd ${WK}
          pwd
          echo ${WK}
          mvn clean package
          '''
        }
      }
    }
    stage('Deploy') {
      when {
        changeRequest()
      }
      steps {
        script {
          sh '''
          ssh -i ~/.ssh/deploy root@192.168.1.165 "mkdir -p /data/app/server-sdk/"
          scp -i ~/.ssh/deploy /var/lib/jenkins/workspace/server-sdk-java/target/server-sdk-java-3.0.4.jar root@192.168.1.165:/data/app/server-sdk/
          ssh -i ~/.ssh/deploy root@192.168.1.165 "cd /data/app/server-sdk/ && nohup java -jar /data/app/server-sdk/server-sdk-java-3.0.4.jar &"
          '''
        }
      }
    }
  }
  post {
    success {
        emailext (
          subject: "PR-result: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' SUCCESS",
          body: """<!DOCTYPE html>
            <html>
            <head>
            <meta charset="UTF-8">
            </head>
            <body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4" offset="0">
                <table width="95%" cellpadding="0" cellspacing="0" style="font-size: 16pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
                    <tr>
                      <td><br />
                        <b><font color="#0B610B"><font size="6">Build Info</font></font></b>
                        <hr size="2" width="100%" align="center" />
                      </td>
                    </tr>
                    <tr>
                      <td>
                      <ul style="font-size:18px">
                        <li>Build Name>>Branch: ${env.BRANCH_NAME}</li>
                        <li>Result: <span style="color:green"> Successful </span></li>
                        <li>Num: ${BUILD_NUMBER}</li>
                        <li>Commit User: ${env.CHANGE_AUTHOR}</li>
                        <li>Commit Title: ${env.CHANGE_TITLE}</li>
                        <li>Build Url: <a href=${BUILD_URL}>${BUILD_URL}</a></li>
                        <li>Build Log: <a href=${BUILD_URL}console>${BUILD_URL}console</a></li>
                      </ul>
                      </td>
                    </tr>
                </table></font>
            </body>
            </html>""",
          to: "${env.CHANGE_AUTHOR_EMAIL}",
          from: "sqchang@taosdata.com"
        )
    }
    failure {
        emailext (
            subject: "PR-result: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' FAIL",
            body: """<!DOCTYPE html>
            <html>
            <head>
            <meta charset="UTF-8">
            </head>
            <body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4" offset="0">
                <table width="95%" cellpadding="0" cellspacing="0" style="font-size: 16pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
                    <tr>
                      <td><br />
                        <b><font color="#0B610B"><font size="6">Build Info</font></font></b>
                        <hr size="2" width="100%" align="center" />
                      </td>
                    </tr>
                    <tr>
                      <td>
                        <ul style="font-size:18px">
                          <li>Build Name>>Branch: ${env.BRANCH_NAME}</li>
                          <li>Result: <span style="color:red"> Failure </span></li>
                          <li>Num: ${BUILD_NUMBER}</li>
                          <li>Commit User: ${env.CHANGE_AUTHOR}</li>
                          <li>Commit Title: ${env.CHANGE_TITLE}</li>
                          <li>Build Url: <a href=${BUILD_URL}>${BUILD_URL}</a></li>
                          <li>Build Log: <a href=${BUILD_URL}console>${BUILD_URL}console</a></li>
                        </ul>
                      </td>
                    </tr>
                </table></font>
            </body>
            </html>""",
            to: "${env.CHANGE_AUTHOR_EMAIL}",
            from: "sqchang@taosdata.com"
        )
    }
  }
}
