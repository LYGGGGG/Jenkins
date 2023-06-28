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

node {
   stage('拉取代码') {
        checkout scmGit(branches: [[name: "*/${branch}"]], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_url}"]])
        }

   stage('代码审查') {
        //定义当前Jenkins的SonarQubeScanner工具
        def scannerHome = tool 'sonar-scanner'
            //引用当前Jenkins SonarQube环境
            withSonarQubeEnv('sonarqube') {
                sh """
                    cd ${project_name}
                    ${scannerHome}/bin/sonar-scanner
                """
            }
        }

   stage('编译，安装公共子工程') {
            sh "mvn -f tensquare_common clean install"
        }

   stage('编译，打包微服务，构建镜像') {
            sh "mvn -f ${project_name} clean package dockerfile:build"
        }

}