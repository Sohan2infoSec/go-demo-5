import java.text.SimpleDateFormat

pipeline {
  options {
    buildDiscarder logRotator(numToKeepStr: '5')
    disableConcurrentBuilds()
  }
  agent {
    kubernetes {
      cloud "go-demo-5-build"
      label "go-demo-5-build"
      serviceAccount "build"
      yamlFile "KubernetesPod.yaml"
    }      
  }
  environment {
    image = "nhatsangvn/go-demo-5"
    project = "go-demo-5"
    domain = "61.28.237.12.nip.io"
    cmAddr = "cm.61.28.237.12.nip.io"
  }
  stages {
    stage("build") {
      steps {
        container("golang") {
          script {
            currentBuild.displayName = new SimpleDateFormat("yy.MM.dd").format(new Date()) + "-${env.BUILD_NUMBER}"
          }
          k8sBuildGolang("go-demo")
        }
        container("docker") {
          k8sBuildImageBeta(image, false, 'DOCKER_HOST="tcp://172.17.0.1:33333"')
        }
      }
    }
    stage("func-test") {
      steps {
        container("helm") {
          k8sUpgradeBeta(project, domain, "--set replicaCount=2 --set dbReplicaCount=1")
        }
        container("kubectl") {
          k8sRolloutBeta(project)
        }
        container("golang") {
          k8sFuncTestGolang(project, domain)
        }
      }
      post {
        always {
          container("helm") {
            k8sDeleteBeta(project)
          }
        }
      }
    }
    stage("release") {
      when {
          branch "master"
      }
      steps {
        container("docker") {
          k8sPushImage(image, false, 'DOCKER_HOST="tcp://172.17.0.1:33333"')
        }
        container("helm") {
          k8sPushHelm(project, "", cmAddr, true, true)
        }
      }
    }
  }
}
