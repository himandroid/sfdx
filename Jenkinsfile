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

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    // withHome.groovy

/**
 * Wrap given closure in a <code>withEnv</code> setting common sfdx env vars to the dir given by <code>home</code>
 * 
 * Home default value is env.WORKSPACE and if both are null the withHome does not map env vars.
 * 
 * <br>Env Vars assigned to home
 * 
 * <li>HOME
 * <li>APPDATA
 * <li>USERPROFILE
 * <li>XDG_DATA_HOME
 * </br>
 * @param home
 * @param additions - list of additional env vars to apply to doWithHome
 * @param doWithHome
 * @return
 */
def call(home = null, additions = [], Closure doWithHome) {
    if (doWithHome == null) {
        error("withHome doWithHome closure cannot be null")
    }
    def useThisHome = home ?: env.WORKSPACE
    def envParams = (useThisHome != null) ? [
            "HOME=${useThisHome}",
            "APPDATA=${useThisHome}",
            "USERPROFILE=${useThisHome}",
            "XDG_DATA_HOME=${useThisHome}"
    ] : []
    debug "home: ${home}, env.WORKSPACE: ${env.WORKSPACE}, useThisHome: ${useThisHome}, withHome envParams: ${envParams}"
    envParams.addAll(additions)
    return withEnv(envParams) { doWithHome.call(); }
}
withHome(["HOME=${env.WORKSPACE}"]) {
    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Deploye Code') {
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }

			println rc
			
			// need to pull out assigned username
			if (isUnix()) {
				rmsg = sh returnStdout: true, script: "${toolbelt} force:mdapi:deploy -d manifest/. -u ${HUB_ORG}"
			}else{
			   rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:mdapi:deploy -d manifest/. -u ${HUB_ORG}"
			}
			  
            printf rmsg
            println('Hello from a Job DSL script!')
            println(rmsg)
        }
    }
}
}
