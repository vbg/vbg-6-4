pipeline {
  agent {
     node {
        label 'maven'
     }
  }
  stages {
     stage('Tests') {
            steps {
              sh './mvnw clean test'
            }
     }
     stage('Package') {
         steps {
              sh '''
               ./mvnw package -DskipTests -Dquarkus.package.type=uber-jar
              '''
              archiveArtifacts 'target/*.jar'
         }
     }
     stage('Build Image') {
        environment { QUAY = credentials('vbg-quay') }
        steps {
           sh '''
             ./mvnw quarkus:add-extension -D extensions="kubernetes, container-image-jib"
           '''
           sh '''
             ./mvnw package -DskipTests \
                -Dquarkus.container-image.build=true \
                -Dquarkus.container-image.registry=quay \
                -Dquarkus.container-image.group=$QUAY_USR \
                -Dquarkus.container-image.name=vbg-test-6-4 \
                -Dquarkus.container-image.username=$QUAY_USR \
                -Dquarkus.container-image.password="$QUAY_PSW" \
                -Dquarkus.container-image.push=true
           '''
        }
     }
  }
}
