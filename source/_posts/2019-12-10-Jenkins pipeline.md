---
title: Jenkins pipeline
categories: Jenkins
tags:
  - Jenkins
abbrlink: 29763
date: 2019-12-10 00:00:00
---

> Jenkins 使用流水线方式部署项目，使发布流程更加清晰透明
流水线采用Groovy语言，须安装pipeline插件
以下Demo：

<!--more-->

```groovy
node {
	stage('1.更新') {
	    dingTalk accessToken: 'a2d0c8bc9da56ba02d0b560ase26341f9c9e4e94ddac0dcd8a9097edf75b53a9', imageUrl: '', jenkinsUrl: 'http://jenkins_url:8080', message: '开始发布', notifyPeople: ''
		git credentialsId: '6bbb1441-ab0b-4f91-b43e-cb4db5a174da', url: 'http://gitlab_url/glory.git'
	}

	stage('2.打包') {
		if (fileExists('/root/.jenkins/workspace/glory-online')) {
			dir('/root/.jenkins/workspace/glory-online') {
				sh 'mvn clean install -U -Dmaven.test.skip -P online'
				archiveArtifacts 'glory-web/target/glory.war' # 成品存档文件
			}
		}
	}

	stage('3.同步') {
	    sshPublisher(
	    	publishers: [sshPublisherDesc(
	    		configName: 'ssh96', 
	    		transfers: [sshTransfer(cleanRemote: false, excludes: '', 
	    			execCommand: 'cp /home/admin/auto_deploy/wartemp/glory.war /home/admin/auto_deploy/production/glory/glory-online.war', 
	    			execTimeout: 120000, flatten: false, makeEmptyDirs: false, 
	    			noDefaultExcludes: false, 
	    			patternSeparator: '[, ]+', 
	    			remoteDirectory: 'auto_deploy/wartemp', 
	    			remoteDirectorySDF: false, 
	    			removePrefix: 'glory-web/target/', 
	    			sourceFiles: 'glory-web/target/glory.war')],
	    		usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
	}

	stage('4.发布'){
		sshPublisher(
			publishers: [sshPublisherDesc(
				configName: 'ssh96', 
				transfers: [sshTransfer(cleanRemote: false, excludes: '',
					execCommand: 'sh /home/admin/scripts/deploy_glory_online.sh', 
					execTimeout: 1200000, flatten: false, makeEmptyDirs: false, 
					noDefaultExcludes: false, 
					patternSeparator: '[, ]+', 
					remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], 
				usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
				
		dingTalk accessToken: 'a2d0c8bc9da56ba02d0b560ase26341f9c9e4e94ddac0dcd8a9097edf75b53a9', imageUrl: '', jenkinsUrl: 'http://jenkins_url:8080', message: '发布完成', notifyPeople: ''
	}
}
```