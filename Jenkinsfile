pipeline {
  agent {
      label 'maven'
  }
  stages {
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            def bc = openshift.startBuild("gateway")
            bc.logs('-f')
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        script {
          openshift.withCluster() {
            def dc = openshift.selector("dc", "gateway")
            dc.rollout().latest()
            dc.rollout().status()
          }
        }
      }
    }
  }
}
