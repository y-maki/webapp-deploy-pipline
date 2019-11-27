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
            openshift.withProject("${params.PROJECT}"){
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
            openshift.withProject("${params.PROJECT}"){
              def p1 = "APPLICATION_NAME=${params.APPLICATION_NAME}"
              def p2 = "BASE_IMAGE=${params.BASE_IMAGE}"
              def p3 = "VERSION=${params.VERSION}"
              def p4 = "SOURCE=${params.SOURCE}"
              openshift.set("env", "bc/${params.APPLICATION_NAME}", 
                p1,
                p2,
                p3,
                p4,
                "--overwrite")
              def bc = openshift.selector('bc', "${params.APPLICATION_NAME}")
              def buildSelector = bc.startBuild()
              buildSelector.logs('-f')
              echo "Build Config completed: ${params.APPLICATION_NAME}"
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
              if (openshift.selector("dc", "blue-${params.APPLICATION_NAME}").exists() | openshift.selector("dc", "green-${params.APPLICATION_NAME}").exists()) {
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
              if (openshift.selector("dc", "blue-${params.APPLICATION_NAME}").exists()) {
                def newPod = "green"
                def oldPod = "blue"
                deploy("${newPod}")
                changeRoute("${newPod}", "${oldPod}")
                delete("${oldPod}")
              } else if (openshift.selector("dc", "green-${params.APPLICATION_NAME}").exists()) {
                  def newPod = "blue"
                  def oldPod = "green"
                  deploy("${newPod}")
                  changeRoute("${newPod}", "${oldPod}")
                  delete("${oldPod}")
              } else {
                  def newPod = "blue"
                  deploy("${newPod}")
                  createRoute("${newPod}")
              }
            }
          }
        }
      }
    }
  }
}
def deploy(color){
  openshift.withCluster() {
    openshift.withProject("${params.PROJECT}"){
      echo "New Color is ${color}."
      def p1 = "APPLICATION_NAME=${params.APPLICATION_NAME}"
      def p2 = "PROJECT=${params.PROJECT}"
      def p3 = "REPLICA_COUNT=${params.REPLICA_COUNT}"
      def p4 = "CONTAINER_IMAGE=${params.CONTAINER_IMAGE}"
      def p5 = "COLOR=${color}"
      def f = openshift.process("webapp-deploy-template", "-p", p1, p2, p3, p4, p5)
      openshift.apply(f).describe()
      def dc = openshift.selector('dc', "${color}-${params.APPLICATION_NAME}")
      dc.related('pods').untilEach{
        return it.object().status.phase == 'Running'
      }
      def latestDcVersion = openshift.selector("dc", "${color}-${params.APPLICATION_NAME}").object().status.latestVersion
      echo "${latestDcVersion}"
      def rc = openshift.selector("rc", "${color}-${params.APPLICATION_NAME}-${latestDcVersion}")
      timeout(10) {
        rc.untilEach(1){
          def rcMap = it.object()
          echo "${rcMap}"
          return (rcMap.status.replicas.equals(rcMap.status.readyReplicas))
        }
      }
       echo "${color}-${params.APPLICATION_NAME} is running"
    }
  }
}
def changeRoute(newColor, oldColor){
  openshift.withCluster() {
    openshift.withProject("${params.PROJECT}"){
      openshift.set("route-backends", "${params.APPLICATION_NAME}", "${newColor}-${params.APPLICATION_NAME}=50", "${oldColor}-${params.APPLICATION_NAME}=50")
      echo "Route has been updated."
    }
  }
}
def createRoute(color){
  openshift.withCluster() {
    openshift.withProject("${params.PROJECT}"){
      echo "Run ${color}-${params.APPLICATION_NAME} expose."
      openshift.expose("svc", "${color}-${params.APPLICATION_NAME}", "--name", "${params.APPLICATION_NAME}", "--path", "/traning-app-01/HelloWorld")
      echo "Route has been created."
    }
  }
}
def delete(color){
  openshift.withCluster() {
    openshift.withProject("${params.PROJECT}"){
      echo "Run ${color}-${params.APPLICATION_NAME} delete."
      openshift.delete("dc,sa,svc", "${color}-${params.APPLICATION_NAME}")
      echo "The resource associated with ${color}-${params.APPLICATION_NAME} has been deleted."
    }
  }
}
