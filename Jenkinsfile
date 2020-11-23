@Library('pipelinex@_refactor_github') _

label = "${UUID.randomUUID().toString()}"
git_project = "mlrun"
git_project_user = "mlrun"
git_deploy_user_token = "iguazio-prod-git-user-token"
git_deploy_user_private_key = "iguazio-prod-git-user-private-key"
git_mlrun_ui_project = "ui"

podTemplate(label: "${git_project}-${label}", inheritFrom: "docker-python") {
    node("${git_project}-${label}") {

        common.notify_slack {
            withCredentials([
                    string(credentialsId: git_deploy_user_token, variable: 'GIT_TOKEN')
            ]) {
                def mlrun_github_client = new Githubc(git_project_user, git_project, GIT_TOKEN, env.TAG_NAME, this)
                def ui_github_client = new Githubc(git_project_user, git_mlrun_ui_project, GIT_TOKEN, this)

                mlrun_github_client.releaseCi(true) {
                    container('docker-python') {
                        stage("build ${git_project}/api in dood") {
                            dir("${Githubc.BUILD_FOLDER}/src/github.com/${git_project_user}/${git_project}") {
                                println(common.shellc("MLRUN_VERSION=${mlrun_github_client.tag.docker} echo test"))
                            }
                        }
                    }
                    
                    container('jnlp') {
                        common.conditional_stage('Create mlrun/ui release', "${mlrun_github_client.tag.toString()}" != "unstable") {
                            def source_branch = mlrun_github_client.getReleasecommittish()
                            
                            println("Source branch is: ${source_branch}, using this as source for ${git_project}/${git_mlrun_ui_project}")
                            println("You are responsible to make sure that this branch exists in ${git_project}/${git_mlrun_ui_project}!")

                            if (!source_branch) {
                                error("Could not get source branch from tag")
                            }

                            ui_github_client.createRelease(source_branch, mlrun_github_client.tag.toString(), true)

                            // TODO: is this really necessary??
                            // github.wait_for_release(
                            //         git_mlrun_ui_project,
                            //         git_project_user,
                            //         "${github.TAG_VERSION}",
                            //         GIT_TOKEN
                            // )
                        }
                    }
                }
            }
        }
    }
}
