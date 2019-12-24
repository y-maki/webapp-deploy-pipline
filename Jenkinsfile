pipeline {
  agent any
  environment {
      applicationpipeline {
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
  stages{
    stage('dc-check') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(project){
              if (openshift.selector("dc", "${params.APPLICATION_NAME}-blue").exists() | openshift.selector("dc", "${params.APPLICATION_NAME}-green").exists()) {
                echo "Run DeploymentConfig Update."
              } else {
                  echo "Run DeploymentConfig Create."
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
            openshift.withProject("${params.PROJECT}"){
              // tagを初期化
              def tag = "blue"
              def altTag = "green"
              appName = "${params.APPLICATION_NAME}-${tag}"
              def rt = openshift.selector("route/${params.APPLICATION_NAME}").object()
              // 古いサービス名を取得(json形式のspec.to.nameに現在接続されているサービス名が格納されているため)
              def oldSvc = rt.spec.to.name 
              echo "The old svc is ${oldSvc}"
              // 古いサービス名がAPP名-blueの場合、アプリケーション名をAPP名-greenに変更する
              if ("${oldSvc}" == "${appName}") {
              // tagを更新
                tag = "green"
                altTag = "blue"
              }
              // アプリケーション名を更新
              appName = "${params.APPLICATION_NAME}-${tag}"
              echo "appName is ${appName}"
              deploy("${tag}")
              route("${tag}","${altTag}")
            }
          }
        }
      }
    }
def deploy(color){
  openshift.withCluster() {
    openshift.withProject("${params.PROJECT}"){
      echo "argument is ${color}."
      def p1 = "APPLICATION_NAME=${params.APPLICATION_NAME}"
      def p2 = "PROJECT=${params.PROJECT}"
      def p3 = "REPLICA_COUNT=${params.REPLICA_COUNT}"
      def p4 = "CONTAINER_IMAGE=${params.CONTAINER_IMAGE}"
      def p5 = "COLOR=${color}"
      def f = openshift.process("webapp-deploy-template", "-p", p1, p2, p3, p4, p5)
      openshift.apply(f).describe()
      openshift.rollout("latest", "${params.APPLICATION_NAME}-${color}")
      def dc = openshift.selector('dc', "${params.APPLICATION_NAME}-${color}")
      dc.related('pods').untilEach{
        return it.object().status.phase == 'Running'
      }
      def latestDcVersion = openshift.selector("dc", "${params.APPLICATION_NAME}-${color}").object().status.latestVersion
      def rc = openshift.selector("rc", "${params.APPLICATION_NAME}-${color}-${latestDcVersion}")
      echo "${rc}"
      timeout(10) {
        rc.untilEach(1){
          def rcMap = it.object()
          echo "${rcMap}"
          return (rcMap.status.replicas.equals(rcMap.status.readyReplicas))
        }
      }
       echo "${params.APPLICATION_NAME}-${color} is running"
    }
  }
}
def route(newColor, oldColor){
  openshift.withCluster() {
    openshift.withProject("${params.PROJECT}"){
      if (openshift.selector("route", "${params.APPLICATION_NAME}").exists()) {
        echo "Change Route settings."
        openshift.set("route-backends", "${params.APPLICATION_NAME}", "${params.APPLICATION_NAME}-${newColor}=10", "${params.APPLICATION_NAME}-${oldColor}=90")
        echo "Route has been updated."
      } else {
          echo "The route did not exist,Create a new Route."
          echo "Run ${params.APPLICATION_NAME}-${oldColor} expose."
          openshift.expose("svc", "${params.APPLICATION_NAME}-${newColor}", "--name", "${params.APPLICATION_NAME}", "--path", "/traning-app-01/HelloWorld")
          echo "Route has been created."
      }
    }
  }
}
def delete(color){
  openshift.withCluster() {
    openshift.withProject("${params.PROJECT}"){
      echo "Run ${color}-${params.APPLICATION_NAME} delete."
      openshift.delete("dc,sa,svc", "${params.APPLICATION_NAME}-${color}")
      echo "The resource associated with ${params.APPLICATION_NAME}-${color} has been deleted."
    }
  }
}Name = 'traning-app-01'
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
