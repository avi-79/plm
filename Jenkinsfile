#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY
    def toolbelt = tool 'toolbelt'

    

 stage('Checkout Source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
		
    }
 stage('Souce Code Analysis'){
	 
 rc=command "sonar-scanner \
  -Dsonar.projectKey=salesforce-DX \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login="
 }	
 withEnv(["HOME=${env.WORKSPACE}"]) {
        
       
	withCredentials([file(credentialsId: JWT_CRED_ID_DH, variable: 'server_key_file')]) {
		
		stage('Authorize DevHub') {   
     rc = bat returnStatus: true, script: "\"${toolbelt}\" auth:jwt:grant --instanceurl ${SFDC_HOST_DH} --clientid ${CONNECTED_APP_CONSUMER_KEY_DH} --username ${HUB_ORG_DH} --jwtkeyfile ${JWT_CRED_ID_DH} --setdefaultdevhubusername"
    if (rc != 0) {error 'Salesforce dev hub org authorization failed.'}
  }
        
  
		
		stage('Create Test Scratch Org') {
    rc = bat "${toolbelt}/sfdx force:org:create --targetdevhubusername HubOrg --setdefaultusername --definitionfile config/project-scratch-def.json --setalias ciorg --wait 10 --durationdays 1"
    println(rmsg)
            def jsonSlurper = new JsonSlurperClassic()
            def robj = jsonSlurper.parseText(rmsg)
            if (robj.status != 0) { error 'org creation failed: ' + robj.message }
            SFDC_USERNAME=robj.result.username
            println(SFDC_USERNAME)
            robj = null
}

    	stage('Set Default Scratch Org') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:config:set --global defaultusername=${SFDC_USERNAME} --json"
            if (rc != 0) { error 'Default scratch org failed' }
        }

        stage('Create password for scratch org') {
 			rc = bat returnStdout: true, script: "\"${toolbelt}\" force:user:password:generate --json"
			println(rmsg)
			def jsonSlurper = new JsonSlurperClassic()
			def robj = jsonSlurper.parseText(rmsg)
            if (robj.status != 0) { error 'password generation failed: ' + robj.message }
            robj = null
        }
	
        stage('Push To Scratch Org') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
            if (rc != 0) { error 'Push failed'}	
            // assign permset
            rc = sh returnStatus: true, script: "\"${toolbelt}\" force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname purealoe"
            if (rc != 0) { error 'permset:assign failed'}
        }
}
}

}
