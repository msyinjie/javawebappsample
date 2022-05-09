import groovy.json.JsonSlurper
import com.cloudbees.plugins.credentials.CredentialsNameProvider
import com.cloudbees.plugins.credentials.Credentials
import com.cloudbees.plugins.credentials.CredentialsProvider
import com.cloudbees.plugins.credentials.common.StandardUsernamePasswordCredentials
import com.cloudbees.jenkins.plugins.sshcredentials.SSHUserPrivateKey

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=f94c2897-a974-41ec-b956-2a349bf945c8',
        'AZURE_TENANT_ID=fbda558f-94a0-4672-819d-19569f5764ae']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'acf-rg0'
      def webAppName = 'jenkinsapp01'
      // login Azure
   

      withCredentials([azureServicePrincipal(credentialsId: 'azure', subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID', clientSecretVariable: 'AZURE_CLIENT_SECRET', clientIdVariable: 'AZURE_CLIENT_ID', tenantIdVariable: 'AZURE_TENANT_ID')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
