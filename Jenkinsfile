import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node ('Duesseldorf-Jenkins-2'){
  stage('init') {
    checkout scm
  }
   stage('com') {
    def mvnHome = tool name: 'Maven3.5.3', type: 'maven'
    sh "${mvnHome}/bin/mvn -B -DskipTests clean package"
  }

  
  stage('deploy') {
    def resourceGroup = 'ghh-test' 
    def webAppName = 'ghhp3ds-webapp-dev'
    // login Azure
    withCredentials([azureServicePrincipal('p3nethenkins')]) {
      sh '''
        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
        az account set -s $AZURE_SUBSCRIPTION_ID
      '''
    }
    // get publish settings
    def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
    def ftpProfile = getFtpPublishProfile pubProfilesJson
    // upload package
    sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/calculator.war -u '$ftpProfile.username:$ftpProfile.password'"
    // log out
    sh 'az logout'
  }
}
