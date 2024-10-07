# Continuous Integration (CI), Continuous Delivery (CD), and Continuous Deployment (CD) 
They are essential practices in modern software development and DevOps. They aim to automate, streamline, and  improve the quality of the development process by shortening the feedback loop and enabling faster, more reliable releases.

## What is Continuous Integration?
Continuous Integration (CI) is a DevOps software development practice that enables the developers to merge their code changes in the central repository. That way, automated builds and tests can 
be run. The amendments by the developers are validated by creating a built and running an automated test against them. 
In the case of Continuous Integration, a tremendous amount of emphasis is placed on testing automation to check on the application. This is to know if it is broken whenever new commits are 
integrated into the main branch.

## What is Continuous Delivery?
Continuous Delivery (CD) is a DevOps practice that refers to the building, testing, and delivering improvements to the software code. The phase is referred to as the extension of the Continuous 
Integration phase to make sure that new changes can be released to the customers quickly in a substantial manner. 
This can be simplified as, though you have automated testing, the release process is also automated, and any deployment can occur at any time with just one click of a button.
Continuous Delivery gives you the power to decide whether to make the releases daily, weekly, or whenever the business requires it. The maximum benefits of Continuous Delivery can only be 
yielded if they release small batches, which are easy to troubleshoot if any glitch occurs.

## What is Continuous Deployment?
When the step of Continuous Delivery is extended, it results in the phase of Continuous Deployment. Continuous Deployment (CD) is the final stage in the pipeline that refers to the automatic 
releasing of any developer changes from the repository to the production. 
Continuous Deployment ensures that any change that passes through the stages of production is released to the end-users. There is absolutely no way other than any failure in the test 
that may stop the deployment of new changes to the output. This step is a great way to speed up the feedback loop with customers and is free from human intervention.

## Summary of the Workflow:
1. Code is Committed → Trigger Continuous Integration (CI): Automated tests, builds, and code quality checks are performed.
2. Tested and Built Code → Proceed to Continuous Delivery (CD): If the code passes the CI pipeline, it is packaged and can be deployed to staging or pre-production environments. At this stage, it is ready for production but may require manual approval.
3. Production Deployment → Continuous Deployment (CD): If automated, the system deploys the software directly to production without manual intervention.





