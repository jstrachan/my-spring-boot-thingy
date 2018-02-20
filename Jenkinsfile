
pipeline {
  agent {
    label "jenkins-maven"
  }

  environment {
    ORG 		= 'jenkinsx'
    APP_NAME    = 'my-spring-boot-thingy'
    GIT_CREDS = credentials('jenkins-x-git')
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
  }

  stages {

    stage('Build Release') {
      steps {
        container('maven') {
          // ensure we're not on a detached head
          sh "git checkout master"

          // until we switch to the new kubernetes / jenkins credential implementation use git credentials store
          sh "git config --global credential.helper store"

          // so we can retrieve the version in later steps
          sh "echo \$(jx-release-version) > VERSION"
          sh "mvn versions:set -DnewVersion=\$(jx-release-version)"
        }

        dir ('./charts/my-spring-boot-thingy') {
          container('maven') {
            sh "make tag"
          }
        }

        container('maven') {
          sh 'mvn clean deploy'
          sh "docker build -f Dockerfile.release -t $JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION) ."
          sh "docker push $JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION)"
        }
      }
    }
    stage('Promote to Environment(s)') {
      steps {
        dir ('./charts/my-spring-boot-thingy') {
          container('maven') {

            sh 'make release'
            sh 'GIT_USERNAME=$GIT_CREDS_USR GIT_API_TOKEN=$GIT_CREDS_PSW jx promote -b --all-auto --timeout 1h --version \$(cat ../../VERSION)'
          }
        }
      }
    }
  }
}
