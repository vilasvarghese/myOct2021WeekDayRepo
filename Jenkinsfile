node {
    
    def mvnHome = tool 'Maven3'


    stage ("Checkout") {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '32c69f66-f8ed-44c3-a327-45a09b6401f8', url: 'https://github.com/akannan1087/myOct2021WeekDayRepo']]])
    }
    
    stage ("Build") {
        sh "${mvnHome}/bin/mvn clean install -f MyWebApp/pom.xml"
    
    }
    stage ("code quality") {
        withSonarQubeEnv('SonarQube') {
            sh "${mvnHome}/bin/mvn sonar:sonar -f MyWebApp/pom.xml"
        }
    }
    
    stage ("Nexus") {
        nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: 'd5b3612f-6b13-4231-a86e-1fde19c2e1d7', groupId: 'com.dept.app', nexusUrl: 'ec2-3-144-209-200.us-east-2.compute.amazonaws.com:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
    }
    
    stage ("Dev Deploy") {
        deploy adapters: [tomcat9(credentialsId: 'd5bec0c9-2337-40bc-bf2e-34ee67ab3811', path: '', url: 'http://ec2-3-141-20-5.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
    }
    
    stage ("Slack") {
        slackSend channel: 'oct-2021-weekend-batch', message: 'DEV Deployment was successful, please start testing'
    }
    
    stage ('DEV Approve')  {
            echo "Taking approval from DEV Manager for QA Deploy"     
            timeout(time: 7, unit: 'DAYS') {
            input message: 'Do you approve to deploy into QA environment?', submitter: 'admin,george.math@email.com,andy.k@email.com'
            }
     }
     
    stage ("QA Deploy") {
        deploy adapters: [tomcat9(credentialsId: 'd5bec0c9-2337-40bc-bf2e-34ee67ab3811', path: '', url: 'http://ec2-3-141-20-5.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
    }
    
    stage ("Slack-QA") {
        slackSend channel: 'qa-testing-team,product-owners-teams', message: 'QA Deployment was successful, please start your functional testing'
    }
    
    stage ('QA Approve')  {
            echo "Taking approval from QA Manager for PROD Deploy"     
            timeout(time: 5, unit: 'DAYS') {
            input message: 'Do you approve to deploy into PROD environment?', submitter: 'admin'
            }
     }
     
    stage ("PROD Deploy") {
        deploy adapters: [tomcat9(credentialsId: 'd5bec0c9-2337-40bc-bf2e-34ee67ab3811', path: '', url: 'http://ec2-3-141-20-5.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
    }
    
    stage ("Slack-final") {
        slackSend channel: 'product-owners-teams', message: 'PROD Deployment was successful, please start your final testing and inform end customers'
    }
}
