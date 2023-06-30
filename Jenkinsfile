//git 凭证ID
def git_auth = "92261813-7f74-49a0-b60f-f6d8dbfc952c"
//git utl
def git_url = "git@192.168.66.100:itheima_group/tensquare_back.git"
//镜像的版本号
def tag = "latest"
//Harbor的url地址
def harbor_url = "192.168.66.102:85"
//镜像仓库中的项目名称
def harbor_project = "tensquare"
//Harbor的登录凭证
def harbor_auth = "4578681b-f055-48d4-8d26-fdfa1aa7a947"

node {
    stage('拉取代码') {
        checkout scmGit(branches: [[name: "*/${branch}"]], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_url}"]])
    }
    // 获取当前项目的名称（这是一个数组）
    def selectedProjectNames = "${project_name}".split(",")
    // 获取当前选择的服务器（服务器集群）
    def selectedServers = "${publish_server}".split(",")

    /*stage('代码审查') {
        for (int i = 0; i < selectedProjectNames.length; i++) {
            //tensquare_eureka_server@10086
            def projectInfo = selectedProjectNames[i];
            def currentProjectName = "${projectInfo}".split("@")[0];
            def currentProjectPort = "${projectInfo}".split("@")[1];
            //定义当前Jenkins的SonarQubeScanner工具
            def scannerHome = tool 'sonar-scanner'
            //引用当前Jenkins SonarQube环境
            withSonarQubeEnv('sonarqube') {
                sh """
                    cd ${currentProjectName}
                    ${scannerHome}/bin/sonar-scanner
                """
            }
        }
    }*/

    stage('编译，安装公共子工程') {
        sh "mvn -f tensquare_common clean install"
    }

    stage('编译，打包微服务，构建镜像，上传镜像') {
        // 遍历每个微服务项目
        for (int i = 0; i < selectedProjectNames.length; i++) {
            //tensquare_eureka_server@10086
            def projectInfo = selectedProjectNames[i];
            def currentProjectName = "${projectInfo}".split("@")[0];
            def currentProjectPort = "${projectInfo}".split("@")[1];

            sh "mvn -f ${currentProjectName} clean package dockerfile:build"

            //定义镜像名称
            def imageName = "${currentProjectName}:${tag}"

            //对镜像打上标签
            sh "docker tag ${imageName} ${harbor_url}/${harbor_project}/${imageName}"

            //把项目推送到Harbor
            withCredentials([usernamePassword(credentialsId: "${harbor_auth}", passwordVariable: 'password', usernameVariable: 'username')]) {
                //登录Harbor
                sh "docker login -u ${username} -p ${password} ${harbor_url}"

                //镜像上传（push整个标签-->新镜像）
                sh "docker push ${harbor_url}/${harbor_project}/${imageName}"

                sh "echo '镜像推送成功'"
            }
            //遍历每个服务器，分别部署
            for (int j = 0; j < selectedServers.length; j++) {
                // 获取当前服务器名称
                def currentServerName = selectedServers[j]
                // 加上参数格式: --spring.profiles.active=eureka-server1/eureka-server2
                def activeProfile = "---spring.profiles.active="

                //根据不同的服务器名称来读取不同的Eureka配置信息
                if (currentServerName == "master-server") {
                    activeProfile = activeProfile + "eureka-server1"
                } else if (currentServerName == "slave-server") {
                    activeProfile = activeProfile + "eureka-server2"
                }

                sshPublisher(publishers: [sshPublisherDesc(configName: "${currentServerName}", transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: "/opt/jenkins_shell/deployCluster.sh $harbor_url $harbor_project $currentProjectName $tag $currentProjectPort $activeProfile", execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
    }
}