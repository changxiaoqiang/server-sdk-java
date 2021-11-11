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
          mvn package install -DskipTests=true
          '''
        }
      }
    }
/*     stage('Parallel test stage') {
      //only build pr
      options { skipDefaultCheckout() } 
      when {
        allOf{
            changeRequest()
            not{ expression { env.CHANGE_BRANCH =~ /docs\// }}
          }
        }
      parallel {
        stage('python_1_s1') {
          agent{label " slave1 || slave11 "}
          steps {
            pre_test()
            timeout(time: 55, unit: 'MINUTES'){
              sh '''
              date
              cd ${WKC}/tests
              ./test-all.sh p1
              date'''
            }
            
          }
        }
        stage('python_2_s5') {
          agent{label " slave5 || slave15 "}
          steps {
            
            pre_test()
            timeout(time: 55, unit: 'MINUTES'){
                sh '''
                date
                cd ${WKC}/tests
                ./test-all.sh p2
                date'''
            }
          }
        }
        stage('python_3_s6') {
          agent{label " slave6 || slave16 "}
          steps {     
            timeout(time: 55, unit: 'MINUTES'){       
              pre_test()
              sh '''
              date
              cd ${WKC}/tests
              ./test-all.sh p3
              date'''
            }
          }
        }
        stage('test_b1_s2') {
          agent{label " slave2 || slave12 "}
          steps {     
            timeout(time: 55, unit: 'MINUTES'){       
              pre_test()
              sh '''
                rm -rf /var/lib/taos/*
                rm -rf /var/log/taos/*
                nohup taosd >/dev/null &
                sleep 10
              '''
              sh '''
              cd ${WKC}/tests/examples/nodejs
              npm install td2.0-connector > /dev/null 2>&1
              node nodejsChecker.js host=localhost
              node test1970.js
        cd ${WKC}/tests/connectorTest/nodejsTest/nanosupport
        npm install td2.0-connector > /dev/null 2>&1
              node nanosecondTest.js
              '''

              sh '''
              cd ${WKC}/tests/examples/C#/taosdemo
              mcs -out:taosdemo *.cs > /dev/null 2>&1
              echo '' |./taosdemo -c /etc/taos
              '''
              sh '''
                cd ${WKC}/tests/gotest
                bash batchtest.sh
              '''
              sh '''
              cd ${WKC}/tests
              ./test-all.sh b1fq
              date'''
            }
          }
        }
        stage('test_crash_gen_s3') {
          agent{label " slave3 || slave13 "}
          
          steps {
            pre_test()
            timeout(time: 60, unit: 'MINUTES'){
              sh '''
              cd ${WKC}/tests/pytest
              ./crash_gen.sh -a -p -t 4 -s 2000
              '''
            }
            timeout(time: 60, unit: 'MINUTES'){
              sh '''
              cd ${WKC}/tests/pytest
              rm -rf /var/lib/taos/*
              rm -rf /var/log/taos/*
              ./handle_crash_gen_val_log.sh
              '''
              sh '''
              cd ${WKC}/tests/pytest
              rm -rf /var/lib/taos/*
              rm -rf /var/log/taos/*
              ./handle_taosd_val_log.sh
              '''
            }
            timeout(time: 55, unit: 'MINUTES'){
                sh '''
                date
                cd ${WKC}/tests
                ./test-all.sh b2fq
                date
                '''
            }                     
          }
        }
        stage('test_valgrind_s4') {
          agent{label " slave4 || slave14 "}

          steps {
            pre_test()
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh '''
                cd ${WKC}/tests/pytest
                ./valgrind-test.sh 2>&1 > mem-error-out.log
                ./handle_val_log.sh
                '''
            }     
            timeout(time: 55, unit: 'MINUTES'){      
              sh '''
              date
              cd ${WKC}/tests
              ./test-all.sh b3fq
              date'''
              sh '''
              date
              cd ${WKC}/tests
              ./test-all.sh full example
              date'''
            }
          }
        }
        stage('test_b4_s7') {
          agent{label " slave7 || slave17 "}
          steps {     
            timeout(time: 105, unit: 'MINUTES'){       
              pre_test()
              sh '''
              date
              cd ${WKC}/tests
              ./test-all.sh b4fq
              cd ${WKC}/tests
              ./test-all.sh p4
              '''
              // cd ${WKC}/tests
              // ./test-all.sh full jdbc
              // cd ${WKC}/tests
              // ./test-all.sh full unit
              
            }
          }
        }
        stage('test_b5_s8') {
          agent{label " slave8 || slave18 "}
          steps {     
            timeout(time: 55, unit: 'MINUTES'){       
              pre_test()
              sh '''
              date
              cd ${WKC}/tests
              ./test-all.sh b5fq
              date'''
            }
          }
        }
        stage('test_b6_s9') {
          agent{label " slave9 || slave19 "}
          steps {     
            timeout(time: 55, unit: 'MINUTES'){       
              pre_test()
              sh '''
              date
              cd ${WKC}/tests
              ./test-all.sh b6fq
              date'''
            }
          }
        }
        stage('test_b7_s10') {
          agent{label " slave10 || slave20 "}
          steps {     
            timeout(time: 55, unit: 'MINUTES'){       
              pre_test()
              sh '''
              date
              cd ${WKC}/tests
              ./test-all.sh b7fq
              date'''              
            }
          }
        } 
        stage('arm64centos7') {
          agent{label " arm64centos7 "}
          steps {     
              pre_test_noinstall()    
            }
        }
        stage('arm64centos8') {
          agent{label " arm64centos8 "}
          steps {     
              pre_test_noinstall()    
            }
        }
        stage('arm32bionic') {
          agent{label " arm32bionic "}
          steps {     
              pre_test_noinstall()    
            }
        }
        stage('arm64bionic') {
          agent{label " arm64bionic "}
          steps {     
              pre_test_noinstall()    
            }
        }
        stage('arm64focal') {
          agent{label " arm64focal "}
          steps {     
              pre_test_noinstall()    
            }
        }
        stage('centos7') {
          agent{label " centos7 "}
          steps {     
              pre_test_noinstall()    
            }
        }
        stage('ubuntu:trusty') {
          agent{label " trusty "}
          steps {     
              pre_test_noinstall()    
            }
        }
        stage('ubuntu:xenial') {
          agent{label " xenial "}
          steps {     
              pre_test_noinstall()    
            }
        }
        stage('ubuntu:bionic') {
          agent{label " bionic "}
          steps {     
              pre_test_noinstall()    
            }
        }
        stage('Mac_build') {
          agent{label " catalina "}
          steps {     
              pre_test_mac()    
            }
        }

        stage('build'){
          agent{label " wintest "}
          steps {
            pre_test()
            script{             
                while(win_stop == 0){
                  sleep(1)
                  }
              }
            }
        }
        stage('test'){
          agent{label "win"}
          steps{
            
            catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                pre_test_win()
                timeout(time: 20, unit: 'MINUTES'){
                bat'''
                cd C:\\workspace\\TDinternal\\community\\tests\\pytest
                .\\test-all.bat wintest
                '''
                }
            }     
            script{
              win_stop=1
            }
          }
        }
      }
    } */
  }
  post {
    success {
      when {
        changeRequest()
      }
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
