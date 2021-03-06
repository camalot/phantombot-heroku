#!groovy
import com.bit13.jenkins.*

if(env.BRANCH_NAME ==~ /master$/) {
		return
}


node ("docker") {
	def ProjectName = "phantombot-heroku"
	def slack_notify_channel = null
	def MAJOR_VERSION = 1
	def MINOR_VERSION = 0


	properties ([
		buildDiscarder(logRotator(numToKeepStr: '25', artifactNumToKeepStr: '25')),
		disableConcurrentBuilds(),
		pipelineTriggers([
			pollSCM('H/30 * * * *')
		]),
	])

	env.PROJECT_MAJOR_VERSION = MAJOR_VERSION
	env.PROJECT_MINOR_VERSION = MINOR_VERSION

	env.CI_BUILD_VERSION = Branch.getSemanticVersion(this)
	env.CI_DOCKER_ORGANIZATION = Accounts.GIT_ORGANIZATION
	env.CI_PROJECT_NAME = ProjectName
	currentBuild.result = "SUCCESS"

	def errorMessage = null
	wrap([$class: 'TimestamperBuildWrapper']) {
		wrap([$class: 'AnsiColorBuildWrapper', colorMapName: 'xterm']) {
			Notify.slack(this, "STARTED", null, slack_notify_channel)
			try {
				withCredentials([[$class: 'StringBinding', credentialsId: env.CI_GNTP_CREDENTIAL_ID, variable: 'GNTP_PASSWORD']]) {
					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: env.CI_ARTIFACTORY_CREDENTIAL_ID,
													usernameVariable: 'ARTIFACTORY_USERNAME', passwordVariable: 'ARTIFACTORY_PASSWORD']]) {
						stage ("install" ) {
							env.PB_USERNAME = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_USERNAME")
							env.PB_OWNER = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_OWNER")
							env.PB_OAUTH = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_OAUTH")
							env.PB_CHANNEL = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_CHANNEL")
							env.PB_WEBPASSWORD = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_WEBPASSWORD")
							env.PB_WEBUSER = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_WEBUSER")
							env.PB_WEBAUTH = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_WEBAUTH")
							env.PB_WEBAUTHRO = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_WEBAUTHRO")
							env.PB_YTAUTH = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_YTAUTH")
							env.PB_YTAURHRO = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_YTAURHRO")
							env.PB_APIOAUTH = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_APIOAUTH")
							env.PB_DISCORDCLIENTID = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_DISCORDCLIENTID")
							env.PB_DISCORDTOKEN = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_DISCORDTOKEN")
							env.PB_DATARENDERSERVICE_TOKEN = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_DATARENDERSERVICE_TOKEN")
							env.PB_TWITTERACCESSTOKEN = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_TWITTERACCESSTOKEN")
							env.PB_TWITTERCONSUMERKEY = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_TWITTERCONSUMERKEY")
							env.PB_TWITTERCONSUMERSECRET = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_TWITTERCONSUMERSECRET")
							env.PB_TWITTERSECRETTOKEN = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "PB_TWITTERSECRETTOKEN")
							env.HEROKU_AUTH = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "HEROKU_AUTH")
							env.HEROKU_EMAIL = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "HEROKU_EMAIL")
							env.HEROKU_APP = SecretsVault.get(this, "secret/${env.CI_PROJECT_NAME}", "HEROKU_APP")
							env.PORT = 2500
							deleteDir()
							Branch.checkout(this, env.CI_PROJECT_NAME)
							Pipeline.install(this)
						}
						stage ("lint") {
							sh script: "${WORKSPACE}/.deploy/lint.sh -n '${env.CI_PROJECT_NAME}' -v '${env.CI_BUILD_VERSION}' -o '${env.CI_DOCKER_ORGANIZATION}'"
						}
						stage ("build") {
							sh script: "${WORKSPACE}/.deploy/build.sh -n '${env.CI_PROJECT_NAME}' -v '${env.CI_BUILD_VERSION}' -o '${env.CI_DOCKER_ORGANIZATION}'"
						}
						stage ("test") {
							sh script: "${WORKSPACE}/.deploy/test.sh -n '${env.CI_PROJECT_NAME}' -v '${env.CI_BUILD_VERSION}' -o '${env.CI_DOCKER_ORGANIZATION}'"
						}
						stage ("deploy") {
							sh script: "${WORKSPACE}/.deploy/deploy.sh -n '${env.CI_PROJECT_NAME}' -v '${env.CI_BUILD_VERSION}' -o '${env.CI_DOCKER_ORGANIZATION}'"
						}
						stage ('publish') {
							// this only will publish if the incominh branch IS develop
							sh script:  "${WORKSPACE}/.deploy/validate.sh -n '${env.CI_PROJECT_NAME}' -v '${env.CI_BUILD_VERSION}' -o '${env.CI_DOCKER_ORGANIZATION}'"

							Branch.publish_to_master(this)
							Pipeline.publish_buildInfo(this)
						}
					}
				}
			} catch(err) {
				currentBuild.result = "FAILURE"
				errorMessage = err.message
				throw err
			}
			finally {
				Pipeline.finish(this, currentBuild.result, errorMessage)
			}
		}
	}
}
