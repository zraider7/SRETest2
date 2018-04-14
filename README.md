# SRETest
Here is a simplified version of the workflow:
GitHub repo update -> AWS CodePipeline {Pulls from source (GitHub repo) -> AWS CodeBuild -> AWS CodeDeploy} -> static website updated

Quick Summary:
You first push the changes to the repo linked in the first question. AWS CodePipeline will then pick up the change to the repo automatically by doing frequent checks to the repo. Once it sees that the repo has been updated, it will initiate a pull from the repo. Once the pull is complete, 
it will start deploying the test environment in AWS CodeBuild using the buildspec.yml file to build the environment (an Ubuntu instance) to a customized state that I defined for my static website. The yaml has the server update, install apache2 and turn it on. I have Codebuild checking the 
code in an environment set up for PHP (even though the index file is just plain HTML). If any errors are caught during CodeBuild, I have it set to stop the Continuous Integration process and email me the error of what happened. If the CodeBuild finished successfully, then we push it to AWS 
CodeDeploy. CodeDeploy spins up the same environment one by one across three Linux systems. If one of the instances fail when being deployed, it will stop the CD process, send me an alert that the job has failed and roll back to the last stable version of the app, if possible. If CodeDeploys 
job completes without issue, I am sent a notification as well. For added availability, the three Linux servers are on a load balancer, so if any of them go down for whatever reason, you will still see the site. Overall, it will take about 4 minutes for your change to reflect to the website.
Below I will document how this pipeline works.

Github:
This was selected since it is the standard to have public repos stored here. Also has great cohesive capabilities between other third party service, such as AWS! Other than accepting changes, in this workflow it does not interact with any third part programs. Just the container for code.

AWS CodePipeline:
Since I wanted to do this pipeline with the least amount of complexity, 3rd party handoffs with confidential information (like AWS IAM keys) and the best cohesion among applications, I chose AWS CodePipeline as my ideal candidate.
CodePipeline pull