# SRETest
Here is a simplified version of the workflow:
GitHub repo update -> AWS CodePipeline {Pulls from source (GitHub repo) -> AWS CodeBuild(-> SNS message) -> AWS CodeDeploy(-> SNS message)} -> static website updated

Quick Summary:
You first push the changes to the repo linked in the first question. AWS CodePipeline will then pick up the change to the repo automatically by doing frequent checks to the repo. Once it sees that the repo has been updated, it will initiate a pull from the repo. Once the pull is complete, 
it will start deploying the test environment in AWS CodeBuild using the buildspec.yml file to build the environment (an Ubuntu instance) to a customized state that I defined for my static website. The yaml has the server update, install apache2 and turn it on. I have Codebuild checking the 
code in an environment set up for PHP (even though the index file is just plain HTML). If any errors are caught during CodeBuild, I have it set to stop the Continuous Integration process and email me the error of what happened. If the CodeBuild finished successfully, then we push it to AWS 
CodeDeploy. CodeDeploy spins up the same environment one by one across three Linux systems. If one of the instances fail when being deployed, it will stop the CD process, send me an alert that the job has failed and roll back to the last stable version of the app, if possible. If CodeDeploys 
job completes without issue, I am sent a notification as well. For added availability, the three Linux servers are on a load balancer, so if any of them go down for whatever reason, you will still see the site. Overall, it will take about 4 minutes for your change to reflect to the website.
Below I will document how this pipeline works.

Github:
This was selected since it is the standard to have public repos stored here. Also has great cohesive capabilities between other third party service, such as AWS! Other than accepting changes, in this workflow it does not interact with any third part programs. Just the container for code.

AWS CodeDeploy:
This is the continuous deployment tool that I chose for the project. I created the application (the static website) and set it to deploy to a group of EC2 instances (a deployment group). When a deployment is underway, it deploys in an "In-place deployment" style, so that the boxes all won'take
go down at once and be respun. This is also to insure availability! The deployment is set to occur on one application at a time in. If something bad happens, it will stop with that server and will attempt to roll back. The load balancer will still display static website from the other two instances.
After that was set up, I created a server role for CodeDeploy to access the instances and deploy the code. Note that the codeDeploy needs a yml file to set up the environment as well, just like CodeBuild. This one is called appspec.yml. It uses the files located in "scripts" to update, install
apache and keep it on. It has slightly different syntax from buildspec.yml, but that is because of the different package managers. It effectively does the same thing that buildspec.yml does. 

AWS CodeBuild:
This tool is used to test the code and that the configurations that you specify to the server will go through without a hitch. I have this configured with AWS codePipeline, so codePipeline will provide the code for it to use and I currently have CodeBuild
run for a linux environment and check PHP code for errors. Although there is no PHP, since this is just a simple website. Using buildspec.yml, it builds out the linux environment to my preference to run the application. I have it set to send me the deployment
status of the app if it succeeds or fails.

AWS CodePipeline:
Since I wanted to do this pipeline with the least amount of complexity, 3rd party handoffs with confidential information (like AWS IAM keys) and the best cohesion among applications, I chose AWS CodePipeline as my ideal candidate for the Continuous Integration tool.
CodePipeline checks the github repo periodically for changes. This is done natively, by you selecting the option to hook up to your github account and then selecting your repo (You need to be logged into github for this to work). Then
once the pull is complete, it will go through codeBuild to test the code , then use codeDeploy to deploy the code if all goes well.

Side stuff
AWS SNS:
This is used to create an SNS and had myself subscribe to it. Used this in order for the CodePipeline to message me on the deployment states of my epplication and where in the CI job we are.
 
 -Dom
