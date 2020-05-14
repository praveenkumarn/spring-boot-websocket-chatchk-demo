pipeline {
agent any
stages {

	stage ('DS-Build-REP-Checkout') {
	    steps {
	        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '7efa7909-5ad8-4648-ab7f-153e0e66a285', url: 'https://github.com/praveenkumarn/spring-boot-websocket-chatting-demo/']]]) 
	}
	}
	stage ('DS-Build-REP-CI') {
	    steps {
		// Maven build step
	        withMaven(maven: 'M2_HOME') { 
	                                       sh "mvn test org.codehaus.mojo:cobertura-maven-plugin:2.7:cobertura -Dcobertura.report.format=xml"
 			 		       sh "mvn clean package" 
			 		     } 
 		} 
	}
stage ('DS-Build-REP-Deploy') {
    steps {
 sshPublisher(publishers: [sshPublisherDesc(configName: 'Docker Swarm Manager', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ls -lrt', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '*,src/**/*'), sshTransfer(cleanRemote: false, excludes: '', execCommand: '''#!/bin/bash
pwd
id
tym=$(date +"%m-%d-%Y-%H-%M")
echo $tym

docker service rm spring-boot-websocket-chat-demo-svc
docker network rm -f epicnetwork

docker stop $(docker ps -a -q)
docker rm -f $(docker ps -a -q)

#docker rm --force 172.31.44.24/spring-boot-websocket-chat-demo:latest_$tym
#docker rmi spring-boot-websocket-chat-demo:latest_$tym

#docker stop $(docker ps | grep ":8020" | awk \'{print $1}\') 
docker images | egrep "latest|SNAPSHOT" | awk \'{print $1 ":" $2}\' | xargs docker rmi -f
docker network ls | grep overlay 

docker network create -d overlay epicnetwork
docker network ls | grep overlay 
docker build -t spring-boot-websocket-chat-demo .
docker tag spring-boot-websocket-chat-demo:latest 172.31.44.24:8083/spring-boot-websocket-chat-demo:latest_$tym

cat ~/passwd.txt | docker login 172.31.44.24:8083 -u admin --password-stdin
docker push 172.31.44.24:8083/spring-boot-websocket-chat-demo:latest_$tym

docker pull 172.31.44.24:8083/spring-boot-websocket-chat-demo:latest_$tym

docker service create --replicas 2  --network epicnetwork --name spring-boot-websocket-chat-demo-svc --publish 8020:8080  172.31.44.24:8083/spring-boot-websocket-chat-demo:latest_$tym
#docker run -d -p 8010:8080  --name spring-boot-websocket-chat-demo-svc 172.31.44.24:8083/spring-boot-websocket-chat-demo:latest_$tym

docker container ls -a
docker service ls
docker service ps spring-boot-websocket-chat-demo-svc

docker service logs spring-boot-websocket-chat-demo-svc 
''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/home/devops/swarm', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
}
	}
}
}
