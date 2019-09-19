pipeline {
  agent none
  environment {
    GITHUB = credentials('github_fint_jenkins')
  }
  stages {
    stage('Generate Model') {
      agent { 
        label 'docker'
      }
      when {
        tag pattern: "v\\d+\\.\\d+\\.\\d+(-\\w+-\\d+)?", comparator: "REGEXP"
      }
      steps {
        sh "docker run -i -v \$PWD:/src fint/fint-model --repo fint-informasjonsmodell-pwfa --filename FINT-pwfa.xml --tag ${TAG_NAME} generate --resource"
        stash(name: 'java', includes: 'java/**')
      }
    }
    stage('Commit Java Model') {
      agent { 
        docker {
          label 'docker'
          image 'fint/git:latest'
        }
      }
      when {
        tag pattern: "v\\d+\\.\\d+\\.\\d+(-\\w+-\\d+)?", comparator: "REGEXP"
      }
      steps {
        script {
          VERSION = TAG_NAME[1..-1]
        }
        git 'https://github.com/FINTmodels/fint-pwfa-model.git'
        sh 'git clean -fdx'
        unstash 'java'
        sh 'rm -rf src/main/java/no/fint/model/*'
        sh 'mv java/* src/main/java/no/fint/model/'
        sh "echo version=${VERSION} > gradle.properties"
        sh 'git config user.email "jenkins@fintlabs.no"'
        sh 'git config user.name "FINT Jenkins"'
        sh 'git add gradle.properties src/main/java/no/fint/model/'
        sh "git commit -m 'Version ${VERSION}'"
        sh "git push 'https://${GITHUB}@github.com/FINTmodels/fint-pwfa-model.git' master:master"
      }
    }
    stage('Generate Consumer') {
      agent { 
        label 'docker'
      }
      when {
        tag pattern: "v\\d+\\.\\d+\\.\\d+(-\\w+-\\d+)?", comparator: "REGEXP"
      }
      steps {
        sh "docker run -i -v \$PWD:/src fint/fint-consumer --repo fint-informasjonsmodell-pwfa --filename FINT-pwfa.xml --tag ${TAG_NAME} setup --name pwfa --component pwfa --package pwfa"
        stash(name: 'consumer', includes: 'fint-consumer-pwfa/**')
      }      
    }
    stage('Commit Consumer') {
      agent { 
        docker {
          label 'docker'
          image 'fint/git:latest'
        }
      }
      when {
        tag pattern: "v\\d+\\.\\d+\\.\\d+(-\\w+-\\d+)?", comparator: "REGEXP"
      }
      steps {
        script {
          VERSION = TAG_NAME[1..-1]
        }
        git 'https://github.com/FINTlabs/fint-consumer-pwfa.git'
        sh 'git clean -fdx'
        unstash 'consumer'
        sh 'git config user.email "jenkins@fintlabs.no"'
        sh 'git config user.name "FINT Jenkins"'
        sh 'git add .'
        sh "git commit -m 'Version ${VERSION}'"
        sh "git push 'https://${GITHUB}@github.com/FINTlabs/fint-consumer-pwfa.git' master:master"
      }      
    }
  }
}
