pipeline {
    agent any

    environment {
        sonarLogin = '2bab7bf7d5af25e2c2ca2f178af2c3c55c64d5d8'
        harborUser = 'admin'
        harborPassword = 'Harbor12345'
        harborHost = '192.168.11.12:8888'
        harborRepo = 'repository'
    }

    stages {
        stage('拉取Git代码'){
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '$tag']], extensions: [], userRemoteConfigs: [[url: 'http://49.233.115.171:8929/root/lsx.git']]])
            }
        }
        stage('Maven构建代码'){
            steps {
                sh '/var/jenkins_home/maven/bin/mvn clean package -DskipTests'
            }
        }
        stage('SonarQube检测代码'){
            steps {
                sh '/var/jenkins_home/sonar-scanner/bin/sonar-scanner -Dsonar.sources=./ -Dsonar.projectname=${JOB_NAME} -Dsonar.projectKey=${JOB_NAME} -Dsonar.java.binaries=target/ -Dsonar.login=${sonarLogin}'
            }
        }
        stage('制作自定义镜像'){
            steps {
                sh '''cd docker
                mv ../target/*.jar ./
                docker build -t ${JOB_NAME}:$tag .
                '''
            }
        }

        stage('推送自定义镜像'){
            steps {
                sh '''docker login -u ${harborUser} -p ${harborPassword} ${harborHost}
                docker tag ${JOB_NAME}:$tag ${harborHost}/${harborRepo}/${JOB_NAME}:$tag
                docker push ${harborHost}/${harborRepo}/${JOB_NAME}:$tag'''
            }
        }

        stage('通知目标服务器'){
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'centos-docker', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: "/usr/bin/deploy.sh $harborHost $harborRepo $JOB_NAME $tag $port", execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }  
        }
    }
    post {
        success {
            dingtalk (
                robot: 'Jenkins-DingDing',
                type:'MARKDOWN',
                title: "success: ${JOB_NAME}",
                text: ["- 成功构建:${JOB_NAME}项目!\n- 版本:${tag}\n- 持续时间:${currentBuild.durationString}\n- 任务:#${JOB_NAME}"]
            )
        }
        failure {
            dingtalk (
                robot: 'Jenkins-DingDing',
                type:'MARKDOWN',
                title: "fail: ${JOB_NAME}",
                text: ["- 失败构建:${JOB_NAME}项目!\n- 版本:${tag}\n- 持续时间:${currentBuild.durationString}\n- 任务:#${JOB_NAME}"]
            )
        }
    }
}
