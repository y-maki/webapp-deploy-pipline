pipeline {
  agent any
  parameters {
      string(name: 'APPLICATION_NAME', defaultValue: 'traning-app-01', description: '')
      string(name: 'PROJECT', defaultValue: 'training', description: '')
      string(name: 'REPLICA_COUNT', defaultValue: '1', description: '')
      string(name: 'BASE_IMAGE', defaultValue: 'wildfly-centos7', description: '')
      string(name: 'VERSION', defaultValue: 'latest', description: '')
      string(name: 'SOURCE', defaultValue: 'https://github.com/hochi2808/traning-app-01.git', description: '')
      string(name: 'CONTAINER_IMAGE', defaultValue: 'traning-app-01:latest', description: '')
  }
  stages {
    stage('bc-check') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(project){
              if (!openshift.selector("bc", "${params.APPLICATION_NAME}").exists()) {
                echo "Run BuildConfig Create."
            } else {
                echo "Run BuildConfig Update."
            }
          }
        }
      }
    }
  }
    stage('bc-apply') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(project){
              def p1 = "APPLICATION_NAME=${params.APPLICATION_NAME}"
              def p2 = "BASE_IMAGE=${params.BASE_IMAGE}"
              def p3 = "VERSION=${params.VERSION}"
              def p4 = "SOURCE=${params.SOURCE}"
              def f = openshift.process("webapp-s2i-build-template", "-p", p1, p2, p3, p4)
              openshift.apply(f).describe()
              def buildSelector = openshift.selector("bc", "${params.APPLICATION_NAME}").related("builds")
              timeout(5) {
                buildSelector.untilEach {
                  return (it.object().status.phase == "Complete")
                }
              }
              echo "Builds have been completed: ${buildSelector.names()}"
            }
          }
        }
      }
    }
    stage('dc-check') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(project){
              if (!openshift.selector("dc", "${params.APPLICATION_NAME}").exists()) {
                echo "Run DeploymentConfig Create."
            } else {
                echo "Run DeploymentConfig Update."
            }
          }
        }
      }
    }
  }
    stage('dc-apply') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(project){
              def p1 = "APPLICATION_NAME=${params.APPLICATION_NAME}"
              def p2 = "PROJECT=${params.PROJECT}"
              def p3 = "REPLICA_COUNT=${params.REPLICA_COUNT}"
              def p4 = "CONTAINER_IMAGE=${params.CONTAINER_IMAGE}"
              def f = openshift.process("webapp-deploy-template", "-p", p1, p2, p3, p4)
              openshift.apply(f).describe()
              def podSelector = openshift.selector("dc", "${params.APPLICATION_NAME}").related("pods")
              timeout(5) {
                podSelector.untilEach {
                  return (it.object().status.phase == "Running")
                }
              }
              echo "Pods is running: ${podSelector.names()}"
            }
          }
        }
      }
    }
  }
}
