Lab V - CICD Pipeline
-------------------------
This lab will quickly show to implement Pipeline to promote images from dev to qa.

#### Deploy an Application
1. Login WebUI using your assigned username: `user##`
2. click into `user##-dev` Project
3. type `php` in search catalog field
4. Select `PHP` builder
5. Click `Next` on the PHP Builder wizard
6. Enter `myapp` as Application Name
7. Enter `https://github.com/piggyvenus/simplephp.git` as the Git repository.
8. Click `Create`

#### Create Pipeline
1. Click back to `Overview`
2. Click `Add to Project` --> `Import YAML/JSON`
3. Copy following to Paste to the text area.
4. Update `user##` to your username.

```
apiVersion: v1
kind: BuildConfig
metadata:
  name: user##-sample-pipeline
  labels:
    app: jenkins-pipeline-example
    name: user##-sample-pipeline
    template: application-template-sample-pipeline
  annotations:
    pipeline.alpha.openshift.io/uses: '[{"name": "myapp", "namespace": "", "kind": "DeploymentConfig"}]'
spec:
  runPolicy: Serial
  source:
    type: None
  strategy:
    type: JenkinsPipeline
    jenkinsPipelineStrategy:
      jenkinsfile: "node {\n}\n"
```
5. Click `Create`

#### Update CICD Pipeline
1. From the left menu, click `Builds` --> `Pipelines`
2. Click onto `user##-sample-pipeline`
3. Click `Actions` --> `Edit`
4. Copy the follow in the Jenkinsfile area
5. Please update the 'user##' to your username
```
node {
stage 'buildInDev'
openshiftBuild(namespace: 'user##-dev', buildConfig: 'myapp', showBuildLogs: 'true')
stage 'deployInDev'
openshiftDeploy(namespace: 'user##-dev', deploymentConfig: 'myapp')
stage 'Deploy QA'
input 'Promote the Dev image to QA?'
stage 'deployInTesting'
openshiftTag(namespace: 'user##-dev', sourceStream: 'myapp',  sourceTag: 'latest', destinationStream: 'myapp', destinationTag: 'promoteToQA')
sh 'oc project user##-qa'
sh 'oc delete all --all -n user##-qa'
sh 'oc new-app user##-dev/myapp:promoteToQA --allow-missing-imagestream-tags=true'
sh 'oc expose svc myapp'
}
```
6. Save
7. Click `Start Pipeline`
