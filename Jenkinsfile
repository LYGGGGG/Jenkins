//git 凭证ID
def git_auth = "92261813-7f74-49a0-b60f-f6d8dbfc952c"
//git utl
def git_url = "git@192.168.66.100:itheima_group/tensquare_back.git"


node {


   stage('拉取代码') {
   checkout scmGit(branches: [[name: "*/${branch}"]], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_url}"]])
   }

}