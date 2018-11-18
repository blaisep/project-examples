#!/usr/bin/env groovy
// Jenkinsfile for Step on in the Jfrog tutorial

node {
    def server = Artifactory.server SERVER_ID
    def rtGradle = Artifactory.newGradleBuild()
    //Clone example code from GitHub repository
    stage 'Build'
        git url: 'https://github.com/blaisep/project-examples.git', branch: 'orbitera'
    //Configure Artifactroy repository to pull/push artifacts
    stage 'Artifactory configuration'
        rtGradle.tool = GRADLE_TOOL // Tool name from Jenkins configuration
        rtGradle.deployer repo:DEPLOY_REPO, server: server
        rtGradle.resolver repo:'libs-release', server: server
        rtGradle.deployer.addProperty("unit-test", "pass").addProperty("qa-team", "platform", "ui")
        def buildInfo = Artifactory.newBuildInfo()
        buildInfo.env.capture = true
    //Run gradle build
    stage 'Exec Gradle'
        if(CLEAN_REPO == "YES") {
            sh 'rm -rf ~/.gradle/caches'
        }
        /* dir("gradle-examples/4/gradle-example-ci-server/") {
            sh 'sh ./increment.sh'
        } */
        rtGradle.run rootDir: "gradle-examples/4/gradle-example-ci-server/", buildFile: 'build.gradle', tasks: 'clean artifactoryPublish', buildInfo: buildInfo
    //Publish artifacts to Artifactory along with build information and scan build artifacts in Xray
    stage 'Publish Build Information & Scan Artifacts'
        shortVCScommit = (buildInfo.vcs.revision.take(8))
        git_history = new File('commits_'${shortVCScommit})
        GIT_COMMIT_HISTORY = sh (
            script: 'git log --oneline > '${git_history},
            returnStdout: true
        ).trim()
        // GIT_HISTORY_FILE = git_history.append()

        server.publishBuildInfo buildInfo
        if (XRAY_SCAN == "YES") {
            def scanConfig = [
                'buildName'      : env.JOB_NAME,
                'buildNumber'    : env.BUILD_NUMBER,
                'failBuild'      : false
            ]
            def scanResult = server.xrayScan scanConfig
            echo scanResult as String
         }
}
