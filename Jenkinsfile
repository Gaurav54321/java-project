pipeline{
  agent none

  environment {
    MAJOR_VERSION = 1
  }

  stages {
    stage ('say hello'){
      agent any
      steps{
        sayHello 'Awesuome student'
      }
    }
    stage('Unit Test'){
      agent {
        label 'apache'
      }
      steps {
        sh 'ant -f test.xml -v'
        junit 'reports/result.xml'
      }
    }
    stage('build') {
      agent {
        label 'apache'
      }
      steps{
      sh 'ant -f build.xml -v'
      }
      post {
        success {
          archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
        }
      }
    }
    stage('deploy'){
      agent {
        label 'apache'
      }
      steps {
        sh "if ! [ -d '/var/www/html/rectangles/all/${env.BRANCH_NAME}' ]; then mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}; fi"
        sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
      }
    }
    stage ("Running on Centos") {
      agent {
        label 'CentOS'
      }
      steps {
        sh "wget http://gauravrawat24032.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
      }
    }
    stage ("Test on Debian") {
      agent {
        label 'apache'
      }
      steps {
        sh "wget http://gauravrawat24032.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 8 9"
      }
    }
    stage ("Promote to green"){
      agent {
        label 'apache'
      }
      when {
        branch 'master'
      }
      steps{
        sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
      }
    }
    stage ('Promte development branch to master') {
      agent {
        label 'apache'
      }
      when {
        branch 'development'
      }
      steps {
        echo "stasing any local chnages"
        sh 'git stash'
        echo "cgecking out development"
        sh 'git checkout development'
        sh 'git pull origin'
        echo "checking out master"
        sh 'git checkout master'
        echo "merging devlopment to master"
        sh 'git merge development'
        echo "pushing to origin master"
        sh 'git push origin master'
        echo 'tagging the release'
        sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
        sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
      }
      post {
        success {
          emailext(
            subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Deve promoted to master",
            body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' success!":</p>
            <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
            to: "cityrocker.101@gmail.com"
            )
        }
      }
    }
  }
    post {
      failure {
        emailext(
          subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!",
          body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Failed!":</p>
          <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
          to: "cityrocker.101@gmail.com"
          )
    }
  }
}
