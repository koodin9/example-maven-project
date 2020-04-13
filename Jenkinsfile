pipeline {
  agent {
    node {
      label 'kubernetes'
    }

  }
  stages {
    stage('Build & Test') {
      agent {
        node {
          label 'kubernetes'
        }

      }
      steps {
        sh 'mvn -Dmaven.test.failure.ignore clean package'
        stash(name: 'build-test-artifacts', includes: '**/target/surefire-reports/TEST-*.xml,target/*.jar')
      }
    }

    stage('Report & Publish') {
      parallel {
        stage('Report & Publish') {
          agent {
            node {
              label 'kubernetes'
            }

          }
          steps {
            unstash 'build-test-artifacts'
            junit '**/target/surefire-reports/TEST-*.xml'
            archiveArtifacts(artifacts: 'target/*.jar', onlyIfSuccessful: true)
          }
        }

        stage('Publish to Artifactory') {
          steps {
            script {
              unstash 'build-test-artifacts'

              def server = Artifactory.server 'Artifactory'
              def uploadSpec = """{
                "files" : [
                  {
                    "pattern": "target/*.jar",
                    "target": "example-repo-local/${BRANCH_NAME}/${BUILD_NUMBER}/"
                  }
                ]

              }"""
              server.upload(uploadSpec)
            }

          }
        }

      }
    }

  }
}
