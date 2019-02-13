pipeline {
  agent {
      label 'maven'
  }
  stages {
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            def bc = openshift.startBuild("gateway-canary")
            bc.logs('-f')
          }
        }
      }
    }
    stage('Deploy Canary') {
      steps {
        script {
          openshift.withCluster() {
            def dc = openshift.selector("dc", "gateway")
            dc.rollout().latest()
            dc.rollout().status()
            sh "oc set route-backends gateway gateway=90 gateway-canary=10"
          }
        }
      }
    }
    stage('Verify Canary') {
      steps {
        script {
             promoteOrRollback = input message: 'Promote or rollback canary deployment?',
                     parameters: [choice(name: "Promote or Rollback?", choices: 'Promote\nRollback', description: '')]
        }
      }
    }
    stage("Rollback canary"){
        when{
            expression {
                return promoteOrRollback == 'Rollback'
            }
        }
        steps{
            echo "Rollback for canary deployment."
            script {
                openshift.withCluster {
                    openshift.withProject('') {
                        openshift.selector('dc', 'gateway-canary').rollout().undo()

                        //wait for rollout
                        openshift.selector('dc', 'gateway-canary').rollout().status()

                        //set canary imagestream back to production tag
                        openshift.tag("coolstore/gateway:latest", "/gateway-canary:latest")
                    }
                }
            }
        }
    }
    stage("Production deployment") {
    when{
        expression {
            return promoteOrRollback != 'Rollback' //Promote or null (first deployment)
        }
    }
    steps {
        script {
            openshift.withCluster() {
                openshift.withProject('') {
                    //Tag latest from build namespace
                    openshift.tag("/gateway-canary:latest", "/gateway:latest")

                    /***
                     * Rollout
                     ***/
                    openshift.selector('dc', 'gateway').rollout().latest()
                    //wait for rollout. It waits until pods are running (if readiness probe is set)
                    openshift.selector('dc', 'gateway').rollout().status()
                    sh "oc set route-backends gateway gateway=100 gateway-canary=0"
                }
            }
        }
      }
    }
  }
}
