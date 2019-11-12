pipeline {
  agent any
  parameters {
      string(name: 'APPLICATION_NAME', defaultValue: 'traning-app-01', description: '')
      string(name: 'PROJECT', defaultValue: 'training', description: '')
      string(name: 'REPLICA_COUNT', defaultValue: '1', description: '')
      string(name: 'BASEI_MAGE', defaultValue: 'wildfly-centos7', description: '')
      string(name: 'VERSION', defaultValue: 'latest', description: '')
      string(name: 'SOURCE', defaultValue: 'https://github.com/hochi2808/traning-app-01.git', description: '')
      string(name: 'CONTAINER_IMAGE', defaultValue: "${APPLICATION_NAME}:${VERSION}", description: '')
  }
  stages {
    stage('bc-check') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(project){
              if (!openshift.selector("bc", applicationName).exists()) {
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
              def p1 = "APPLICATION_NAME=${applicationName}"
              def p2 = "BASE_IMAGE=${baseImage}"
              def p3 = "VERSION=${version}"
              def p4 = "SOURCE=${source}"
              def f = openshift.process("webapp-s2i-build-template", "-p", p1, p2, p3, p4)
              openshift.apply(f).describe()
              def buildSelector = openshift.selector("bc", applicationName).narrow("bc").related("builds")
              timeout(5) {
                buildSelector.untilEach(1) {
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
              if (!openshift.selector("dc", applicationName).exists()) {
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
              def p1 = "APPLICATION_NAME=${applicationName}"
              def p2 = "PROJECT=${project}"
              def p3 = "REPLICA_COUNT=${replicaCount}"
              def p4 = "CONTAINER_IMAGE=${containerImage}"
              def f = openshift.process("webapp-deploy-template", "-p", p1, p2, p3, p4)
              openshift.apply(f).describe()
              def podSelector = openshift.selector("dc", applicationName).narrow("dc").related("pods")
              timeout(5) {
                podSelector.untilEach(1) {
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
