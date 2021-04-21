library identifier: 'WorkflowLibsShared@master', retriever: modernSCM(
	[$class: 'GitSCMSource', remote: 'https://git.balgroupit.com/CICD-DevOps/WorkflowLibsShared.git']
  )

import groovy.json.StringEscapeUtils

String formUrlencode(Map params) {
	params.collect { k,v -> URLEncoder.encode(k, "UTF-8") + "="  + URLEncoder.encode(v, "UTF-8")}.join('&')
}

def deployRestEndPoint(name, auth, env = '') {
	println "deploying $name to $env"
	String url  = "https://${env}jira.baloisenet.com/atlassian/rest/scriptrunner/latest/custom/customadmin/com.onresolve.scriptrunner.canned.common.rest.CustomRestEndpoint"
	String scriptText = filePath("src/RESTEndpoints/$name").readToString()
	String payload = """{"FIELD_INLINE_SCRIPT":"${StringEscapeUtils.escapeJavaScript(scriptText)}","canned-script":"com.onresolve.scriptrunner.canned.common.rest.CustomRestEndpoint"}"""
	http_post(url, auth, payload, 'application/json')
}

def getXsrfToken(auth, env) {
	String url = "http://${env}jira.baloisenet.com:8080/atlassian/secure/admin/EditAnnouncementBanner!default.jspa"
	HttpCookie.parse("Set-Cookie:"+http_head(url, auth)['Set-Cookie'].join(', ')).find{it.name == 'atlassian.xsrf.token'}.value	
}

def deployBanner(auth, env = '', xsrf_token) {
	String scriptText = filePath("src/banner.html").readToString()
	String payload =  formUrlencode([
	                                 atl_token : xsrf_token,
	                                 announcement : scriptText,
	                                 bannerVisibility : 'public',
	                                 "Set+Banner" : "Set+Banner"
	                                	 ])
	String url  = "http://${env}jira.baloisenet.com:8080/atlassian/secure/admin/EditAnnouncementBanner.jspa"
	http_post(url, auth, payload, 'application/x-www-form-urlencoded', ['atlassian.xsrf.token' : xsrf_token])
}

def deployServerScript(scriptPath, auth, env = ''){
	String scriptText = filePath("src/$scriptPath").readToString()
	String url = "http://${env}jira.baloisenet.com:8080/atlassian/rest/scriptrunner/latest/custom/serverScript/$scriptPath"
	println "deploying $scriptPath to $url"
	http_put(url, auth, scriptText)
}

node {
	stage('deploy JIRA test'){		
		checkout scm
		String env = "int-"
		withSecret("auth", "2!f2rGONQdMqyc/qQV/uXwfIHB+iVFumc984DAM6N0pYgaxXRUvTfGDqlf+KoCzoR1", "jenkins admin basic auth creds"){
			//deployRestEndPoint('hostname.groovy',auth,  env)
			//String xsrf_token  = getXsrfToken(auth, env)
			//deployBanner(auth, env, xsrf_token)
			deployServerScript('transitionPostfunctions/CIT_DCS_Change-UpdateValuemation.groovy', auth, env)
		}
	}
}  
