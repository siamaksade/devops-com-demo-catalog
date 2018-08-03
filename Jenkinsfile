pipeline {
  agent {
      label 'maven'
  }
  stages {
    stage('Build JAR') {
      steps {
        git url: 'https://github.com/siamaksade/devops-com-demo-catalog.git'
        sh "cp .settings.xml ~/.m2/settings.xml"
        sh "mvn package"
      }
    }
    stage('Archive JAR') {
      steps {
        sh "mvn deploy -DskipTests"
      }
    }
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(env.DEV_PROJECT) {
              openshift.startBuild("catalog", "--from-file=target/catalog-${readMavenPom().version}.jar", "--wait")
            }
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(env.DEV_PROJECT) {
              def result, dc = openshift.selector("dc", "catalog")
              dc.rollout().latest()
              timeout(10) {
                  result = dc.rollout().status("-w")
              }
              if (result.status != 0) {
                  error(result.err)
              }
            }
          }
        }
      }
    }
  }
}