@Library('test-library@v2') _
node {
    try {

        stage("Init") {
            def scmVars = checkout scm
        }

        def directories = sh(script: "ls ${WORKSPACE}/charts", returnStdout: true).trim().split("\n")

        stage("Validate"){
            def parallelStages = [:]
            for (dir in directories) {
                def dirName = dir
                parallelStages["${dirName}"] = {
                    print(dirName)
                    validate("${WORKSPACE}/charts/${dirName}")
                }
            }
            parallel parallelStages
        }

        if(env.BRANCH_NAME == 'master'){
            stage("Publish"){
                def parallelStages = [:]
                for (dir in directories) {
                    def dirName = dir
                    parallelStages["${dirName}"] = {
                        def changes = sh(script: "git diff --name-only HEAD^ HEAD charts/${dirName}", returnStdout: true).trim()
                        if(changes){
                            publishArtifact("${WORKSPACE}/charts/${dirName}")
                        }
                    }
                }
                parallel parallelStages
            }
        }
    } finally {
        cleanWs() 
    }

}

def validate(dirName){
    dir(dirName){
        withHelmUtil(){
            withPaastryHelmRegistry(){
                sh("helm template test ./ -f values.yaml")
            }
        }
    }
}

def publishArtifact(dirName){
    dir(dirName){
        withHelmUtil(){
            withPaastryHelmRegistry(){
                sh("helm cx publish")
            }
        }
    }
}