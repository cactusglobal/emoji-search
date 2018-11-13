## Continuous Integration setup for React Application emoji-search

This setup will help you configure ci process for react applications.

### Configured Jenkins Server using cloudformation template.
**Steps:** 
* Upload cloudformation template in path automation/cfn-ec2-with-security-group.yaml to AWS cloudformation create stack console to create a server and software instllation including jenkins application 
* Manually Setup jenkins login and plugins. Following plugins have been installed
    * Dry - for fopy paste detection analysis
    * Clover - For Test Coverage
    * Checkstyle - for Code errors and Warnings
    * Static Analysis collector - for showing the build trend for the above used tools
    * Blue Ocean - Provides bettern UI and visualization of Pipeline.
    * Slack Notification - To notify build status on Slack
    * GithubNotify - To indicate build status of a commit hash on Github
    * Junit - Record unit/component test results
    * Role-based Authorization Strategy Plugin
    * CloudBees Docker Build and Publish - To build and publish docker containers
    * S3 Publisher Plugin - To upload artifacts on S3 if required.
* Configured jenkins global credentials to be used with jobs.
* Setup a local image on the jenkins server with name shridharalve/node:reactapp-img using Dockerfile in automation/Dockerfile. Jenkins will use this image to provision containers to run builds.

### Configure Jenkins pipeline

The Jenkins pipeline is configured in Jenkinsfile included in the repository.

It includes the following stages:

* Prepare
  Setup docker agent.
  Notify pending stage to github sha pushed to the repository

* Build
  run npm install to install required application packages
  install CI tools elinst and jscpd for static analysis 
* Static Analysis
  Perform static analysis and collect result using checkstyle and dry plugins
* Test
  Run npm test and collect unit/component testing coverage report
* Pull to test server
  If the build has passed and commit message has substring "[ci pull-test]" pull code to test servers for manual testing.
* Package for production
  Changes merged or pushed on master branch are baked in a docker container and pushed to docker hub registry

* Pipline configured under reactapp-ci and reactapp-ci-multibranch incase we need to run different pipeline for each feature branch.

* Jenkins login to check the pipeline sent in email seperately
  

### Setup github webhooks

  Github webhooks have been setup, so that on every push to the repository shridhars/emoji-search the jekins pipline is triggered to run ci for the git branch

### Setup docker hub registry to publish production ready build images.

* These images contain the master code for the latest github merge or master push.
  If the master branch is updated the reactapp-ci build installs the necessary software and 
  These production ready build images can be provisioned on EC2 instances for deployment. 
  Images are stored and versioned according to the build no.
  eg. a 3rd jenkins build will have number  shridharalve/node:reactapp-3

  Check Tags under shridharalve/node for images


### Git post-commit hook

* The git post commit hook in automation/post-commit can be added to local copy of git repository to run build on every commit and deploy it to the docker container



