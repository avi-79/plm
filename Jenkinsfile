#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG_DH="avinesh17@force.com"
    def SFDC_HOST_DH = "https://login.salesforce.com"
    def JWT_CRED_ID_DH = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY_DH="3MVG9n_HvETGhr3B6A3.exf_Yzlw3t8b7ifS4x8DMPFTXx6tG9fXnE5zdUCuDqVVur950EbWfVi7L4ymGiqO2"
    
    

    def toolbelt = tool 'toolbelt'

    stage('Checkout Source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }
	
 stage('Souce Code Analysis'){
	 
 bat "sonar-scanner.bat \
  -Dsonar.projectKey=salesforce-DX \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=jenkins-sonar"
 }
	
    	stage('Authenticate Devhub') {
            bat "sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY_DH} \
--jwtkeyfile server.key --username ${HUB_ORG_DH} \
--setdefaultdevhubusername --setalias myhuborg"
        }
        
        stage('Create Scratch Org') {
            // need to pull out assigned username
            rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:org:create -f config/project-scratch-def.json --json -s -a QAUbuntu"
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
 			rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:user:password:generate --json"
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
