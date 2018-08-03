pipeline {
  agent {
      label 'maven'
  }
  stages {
    stage('Build JAR') {
      steps {
        sh "mvn package -s ~/.m2/settings.xml"
      }
    }
    stage('Archive JAR') {
      steps {
        sh "mvn deploy -DskipTests -s ~/.m2/settings.xml"
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
                  result = dc.rollout().status()
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