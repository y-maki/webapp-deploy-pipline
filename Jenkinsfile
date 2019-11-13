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
              openshift.apply(f)
              def bcSelector = openshift.selector("bc", "${params.APPLICATION_NAME}")
              def lastBcVersion = bcSelector.object().status.lastVersion
              def buildSelector = openshift.selector("build", "${params.APPLICATION_NAME}-${lastBcVersion}")
              buildSelector.untilEach {
                def builds = it.object()
                echo "${builds}"
                echo "${buildSelector}"
                echo "${lastBcVersion}"
                def a = bcSelector.object().status.lastVersion
                lastBcVersion = "${a}"
                echo "${builds}.status.phase"
                return (builds.status.phase == 'Running')
               }
              bcSelector.logs('-f')
              bcSelector.describe()
              echo "Build Config completed: ${bcSelector.names()}"
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
              def latestDcVersion = openshift.selector("dc", "${params.APPLICATION_NAME}").object().status.latestVersion
              echo "${latestDcVersion}"              
              def rc = openshift.selector("rc", "${params.APPLICATION_NAME}-${latestDcVersion}")
              timeout(10) {
                rc.untilEach(1){
                    def rcMap = it.object()
                    echo "${rcMap}"
                    return (rcMap.status.replicas.equals(rcMap.status.readyReplicas))
                }
              }
              echo "${params.APPLICATION_NAME} is running"
            }
          }
        }
      }
    }
  }
}
