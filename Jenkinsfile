pipeline {
  agent {
     node {
        label 'maven'
     }
  }
  environment {
     RHCT-USER = 'iwrruh'
     DEPLOYMENT-STAGE = 'vbg-shopping-cart-stage'
     DEPLOYMENT-PROD = 'vbg-shopping-cart-production' 
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
                -Dquarkus.container-image.registry=quay.io \
                -Dquarkus.container-image.group=$QUAY_USR \
                -Dquarkus.container-image.name=vbg-test-6-4 \
                -Dquarkus.container-image.username=$QUAY_USR \
                -Dquarkus.container-image.password="$QUAY_PSW" \
                -Dquarkus.container-image.tag=build-${BUILD_NUMBER} \
                -Dquarkus.container-image.push=true
           '''
        }
     }
     stage('Deploy Staging') {
         environment {
           APP_NAMESPACE = 'vbg-shopping-cart-stage'
           QUAY = credentials('vbg-quay')
         }
         steps {
	   sh """
              oc set image deployment ${DEPLOYMENT-STAGE} \
              shopping-cart-stage=quay.io/${QUAY_USR}/vbg-test-6-4:build-${BUILD_NUMBER} \
              -n ${APP_NAMESPACE} --record
           """
         }
     }
  }
}
