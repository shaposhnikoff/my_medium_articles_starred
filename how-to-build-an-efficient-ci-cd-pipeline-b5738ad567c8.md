
# How to build an efficient CI/CD pipeline

This article is based on our experience at Cinglevue in building a CI/CD pipeline for the delivery of Virtuoso the enterprise education platform developed by Cinglevue.

![](https://cdn-images-1.medium.com/max/7558/1*IwRwuzdv-TVIDb4yLHa8Ig.jpeg)

Continuous Integration and Continuous Delivery has been on the top of the agenda for many agile software development teams for the last few years. It was recognized to be the foundation for establishing a DevOps practice which most organisations envisioned to be the key enabler for fast and reliable software delivery.

Continuous Delivery is a core ideology in agile software development. One of the first principles in [Agile Manifesto](https://agilemanifesto.org/principles.html) established way back on 2001 reads as,
> “ Our highest priority is to satisfy the customer through early and continuous delivery of valuable software.”

The success in agile software development strongly depends on the team’s ability to quickly roll out features to the end users and continuously improve the software incorporating the feedback from the end users. Shorter the cycle, better the user satisfaction would be. An efficient CI/CD pipeline would be the key to achieving such quick turnarounds.

## Fundamentals

There are few fundamentals which drives a CI/CD pipeline.

* **Integration and Verification** — in a typical software development setup you expect multiple developers to be developing in their own feature branches which they periodically integrate to a common development branch. When a piece of code is integrated (or even before it’s integrated) , it is essential that a verification step is available that can quickly ensure that the particular integration would not break existing functionality, would not degrade performance or even would not introduce security loopholes.

* **Automation **— in-order to achieve speed, it is essential that verification is automated. i.e. this should be comprised of series of automated tests that would cover most critical aspects of the software and could be executed in a reasonable period of time.

* **DevOps culture **— the development team has a big role to play in ensuring the continuity of the pipeline. For example what happens when a build fails or test fails?. Fixing such issues should take top priority or otherwise it will diminish the returns of the CI/CD process.

* **Containerisation **— not mandatory but if the deployment is based on containers, it will reduce the complexity.

## Our Approach

Designing a CI/CD pipeline for delivering an enterprise application requires consideration not only on fundamentals but also on practical challenges specific to the organisation or software. Some points to consider are,

* **Software development process** — CI/CD would produce the best ROI in an agile environment.

* **Unit test coverage **— this is a key piece of CI and if you are low on test coverage, it would make sense to work on that prior to implementing the CI/CD pipeline.

* **Extent of automation** — this would decide whether you can solely depend on automated tests or whether you want to introduce some manual testing also into the process.

* **Nature of the test suite** —the number of test cases or more importantly the time taken to execute the tests may need to be considered. For example if the tests take long to run, then it might not be practical to execute them at each code commit.

In our case we adopted the four step approach outlined below.

![](https://cdn-images-1.medium.com/max/2226/1*-BziHNWo19nQ_edSiO6y0g.png)

Continuous Delivery and Continuous Deployment are often confused of but are two different things. Martin Fowler describe the differences as follows,
> “Continuous Delivery is sometimes confused with Continuous Deployment. **Continuous Deployment** means that every change goes through the pipeline and automatically gets put into production, resulting in many production deployments every day. Continuous Delivery just means that you are able to do frequent deployments but may choose not to do it, usually due to businesses preferring a slower rate of deployment. In order to do Continuous Deployment you must be doing Continuous Delivery.”

Fully automated continuous deployment is often considered a business risk especially in an enterprise setup. This is why a “release process” exists where the changes would be systematically and predictably delivered to the end users.

### Continuous Integration

Our CI process would be triggered when a developer commits the code to their relevant feature branch. The Git hooks associated with Git repository would now trigger the build process in a [Jenkins Cluster](https://jenkins.io/doc/book/architecting-for-scale/). Jenkins pipelines are used to drive the build process and there is a **quality gate check** that is associated with the build process. Quality gate check should be based on what is considered as minimal requirements to commit to the common development branch. In our context the quality gate check validates,

* Whether build is successful

* Unit tests have passed

* No code style violations

* Code coverage on new code is above 80%

* No vulnerabilities or code smells reported by [Sonar](https://www.sonarqube.org/) scan.

### Continuous Delivery

If the quality gate has passed, the developers can submit their pull requests. Integration Managers would merge the code to the common development branch. This would kick start the build process on the common development branch and if successful would go on to build the docker images.

Ideally all tests should execute as a part of the integration process but practically this would be inefficient due to the test execution time. Therefore we’ve designed this into an overnight segment called “Continuous Testing”.

### Continuous Testing (CT)

This is an overnight process where tests such as functional tests, security scans and performance tests are executed on the latest successful build of the software. Prior to test execution the new containers would be deployed in the continuous testing environment based on the latest docker images. The persistent volumes attached to the Kubernetes cluster would be restored as prerequisite for testing. Note that all these activities are scheduled and completely automated.

The test report is examined in the following morning ahead of the daily standup meetings. Any scripting issues would be fixed by the quality assurance team and any code issues would be fixed by the development team. CT failures are considered priority and would be fixed in the earliest instance possible.

### Controlled Deployment

The deployment is simplified as most of the hard work is already done in the three previous steps. A release can be done at any point with a successful CT cycle being the only qualification criteria. The release scripts would,

* Tag the docker images with the relevant version number

* Tag the source repositories with the version number

Now the release can be deployed in other environments in the release pipeline. Ultimately the promotion of the release to the production would be a business decision. A docker + kubernetes setup would simplify the deployment process and the results would be predictable across all environments.

## Available Technology

In our case we choose to use a combination of tools as it seems to provide the best solution for our complicated needs. Most teams developing enterprise products would benefit from such a ground up approach. Our tool stack consists of,

* [Jenkins](https://jenkins.io/doc/book/architecting-for-scale/) in master-slave mode as the build server

* [Jenkins Pipelines](https://jenkins.io/doc/book/pipeline/) to drive the CI process

* [Git Hooks](https://www.atlassian.com/git/tutorials/git-hooks) to trigger builds with code commits

* [SonarQube](https://www.sonarqube.org/) as the code quality tool

* [Robot Framework](https://robotframework.org/) for automating functional tests

* [JMeter](https://jmeter.apache.org/) for performance testing

* [OWASP ZAP](https://www.zaproxy.org/) for security scanning

However there are other commercial and free tools available that you might want to evaluate depending on your requirement.

* T[ravis CI](https://travis-ci.org/)

* [Circleci](https://circleci.com/)

* [Team City](https://www.jetbrains.com/teamcity/)

* [Bamboo](https://www.atlassian.com/software/bamboo)

The version control providers also offer their own stack of CI/CD tools.

* [GitLab CI/CD](https://docs.gitlab.com/ee/ci/)

* [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines)

The cloud vendors also offer CI/CD tools to fast track integration in the cloud

* [AWS CodePipeline](https://aws.amazon.com/codepipeline/)

* [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/)

* [GCP Tools](https://cloud.google.com/docs/ci-cd/)

* [Knative](https://cloud.google.com/knative/) (for serverless)

## Conclusion

An efficient CI/CD pipeline can significantly improve the time to market and help maintain stability and quality of the software being delivered. However a successful implementation requires not only the right technology but also the commitment from the key stakeholders. The project sponsors should take a long term view when investing and the technical leadership has a major role in driving the transformation.
