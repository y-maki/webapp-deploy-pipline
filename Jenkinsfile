pipeline {
  agent any
  environment {
      applicationName = 'traning-app-01'
      project = 'training'
      replicaCount = '1'
      baseImage = 'wildfly-centos7'
      version = 'latest'
      source = 'https://github.com/hochi2808/traning-app-01.git'
      containerImage = 'traning-app-01:latest'
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
            }
          }
        }
      }
    }
  }
}
