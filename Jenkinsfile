@Library('main-shared-library') _

def git_repo_name = scm.getUserRemoteConfigs()[0].getUrl().replaceFirst(/^.*\/([^\/]+?).git$/, '$1')

properties([
	parameters([
		string(name: 'BASE_VERSION', defaultValue: '2.0'),
	])
])

pipeline {
	agent {
		kubernetes {
			yaml kubernetes.base_pod([
				base_image_uri: "534369319675.dkr.ecr.us-west-2.amazonaws.com/sl-jenkins-base-ci:latest",
				ecr_uri: "534369319675.dkr.ecr.us-west-2.amazonaws.com",
				shell_memory_request: "300Mi",
				shell_cpu_request: "0.5",
				shell_memory_limit: "900Mi",
				shell_cpu_limit: "1",
				node_selector: "jenkins"
			])
			defaultContainer 'shell'
		}
	}
	options {
		ansiColor("xterm")
		buildDiscarder(logRotator(numToKeepStr: "20"))
		githubProjectProperty("https://github.com/Sealights/${git_repo_name}/")
	}
	environment {
		TZ = "Asia/Jerusalem"
	}
	stages {
		stage("MongoDB Migration CI") {
			when {
				anyOf {
					changeRequest()
					branch 'master'
				}
			}
			stages {
				stage('Init') {
					steps {
						script {
							env.CURRENT_VERSION = "${params.BASE_VERSION}.${env.BUILD_NUMBER}-${env.BRANCH_NAME}".replace("/", "-")
							env.IS_PR = env.BRANCH_NAME.startsWith('PR-') ? "true" : "false"
							if (env.IS_PR == "false")
								env.CHANGE_BRANCH = env.BRANCH_NAME
							tools.set_npm_registries()
							sh "npm install && npm pack"
						}
					}
				}
				stage('Publish') {
					steps {
						script {
							sh "npm version ${CURRENT_VERSION} --no-git-tag-version"
							if ("${GIT_BRANCH}" == "master") {
								env.CURRENT_VERSION = env.CURRENT_VERSION.replace("-${env.BRANCH_NAME}", "")
								sh "npm publish --tag master"
							} else {
								sh "npm publish --tag " + env.CHANGE_BRANCH.replace('/', '-')
							}
						}
					}
				}
			}
		}
	}
}
