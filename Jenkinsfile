#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME
    def alias="test"
    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def TEST_LEVEL = 'RunLocalTests'

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY

    def toolbelt = tool 'toolbelt'

 



    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
    }

 

    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // ----
      withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('authorizatiom') {
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }


        }
        stage('Create Test Scratch Org') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:org:create --targetdevhubusername avinesh17@force.com --setdefaultusername --definitionfile config/project-scratch-def.json --setalias ${alias} --wait 10 --durationdays 1"
              }
 
        stage('set password for org') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:user:password:generate --targetusername ${alias}"
        }         

                
        stage('Push To Test Scratch Org') {
              rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:push "
              println(rc)
              if (rc != 0) {
              error 'Salesforce push to test scratch org failed.'
                   }
        }

        stage('Run apex test') {
             rc = bat returnStatus: true, script: "\"${toolbelt}\" force:apex:test:run --targetusername ${alias} --wait 10 --resultformat tap --codecoverage --testlevel ${TEST_LEVEL}"
          if (rc != 0) {
                    error 'Salesforce unit test run in test scratch org failed.'
                       }
        }

      stage('generate package')
      {
         rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:convert"

          if (rc != 0) {
                    error 'Failed to generate package'
                       }
      }


        
    }
   
}
