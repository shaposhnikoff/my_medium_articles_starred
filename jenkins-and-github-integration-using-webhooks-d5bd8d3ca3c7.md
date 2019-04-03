
# Jenkins and github integration using webhooks

Reading Time: 2 minutes

![](https://cdn-images-1.medium.com/max/3032/1*b-tu9qys1GqJAKBLHBikvQ.png)

TL;DR

As github has deprecated service integration feature, I had to reconfigure jenkins integration using webhooks. As this is not straightforward to do, this blog post will help you to save some time if you are using github/jenkins integration.

## Precondition

Jenkins github plugin.

## Github Personal Access Token.

Lets go down the rabbit hole. You first need to create github personal access token for user that has access to github repository. In my case, that was my github account.

1. Click on your github avatar

1. Select Settings

1. Select Developer Settings

1. Select Personal Access Token

1. Enter your github password

Hmmm, possible github UX issue, this is 4th level depth to get to this feature. Step 5 shows that github knows web security.

1. Click generate new token

1. Enter meaningful name

1. Select permissions: admin:repo_hook, repo

1. Generate token

1. Copy token. Be careful, this is your only chance to copy it.

Again, proof that github understands web security.

## Jenkins Global credentials

Using github token, you will create jenkins global credential.

1. In jenkins, click credentials

1. Click global (any of provided links)

My note on jenkins UX. Its free, and I understand that team do not have much resources. But credentials UX is really messed up.

1. On left, click Add credentials

1. For kind select secret text

1. For scope leave Global option

1. In secret, paste github token.

1. Enter meaningful description. Trust me, this is very important, otherwise you will not be able to select this credentials.

Jenkins Github settings

1. Manage Jenkins

1. Configure System

1. Scroll down to Github settings

1. Add github server

1. Set meaningful name

1. Leave default API url

1. Select credentials created in previous step.

1. Click apply

1. Click test connection. Following message should appear: Credentials verified for user xxx, rate limit: 4998

Note [rate limit](https://developer.github.com/v3/#rate-limiting). You have 5000 requests per hour for all github integrations, not just Jenkins.

## Github repository webhook

1. Go to github repository that you are integrating with Jenkins. You need to have admin rights for that repo.

1. Click Settings

1. Select webhooks

1. Add webhook

1. Payload url is jenkins_url_with_http_or_https/github-webhook/

1. Content type: application/json

1. Just push event if you only want to trigger jenkins jobs by branch pushes.

1. Check Active

1. Click update

1. Recent deliveries should have green check icon.

If that is not the case, click on latest delivery that will contain HTTP request/response.

## Jenkins job build trigger.

1. Go to your jenkins job

1. In Build Triggers section, check out: GitHub hook trigger for GITScm polling

## Check

1. Commit something to your repo in branch for which job is configured.

1. Job should start executing.

Resources

I used those two in order to decript all steps in proper order:

1. [Cloud Bees webhooks](https://support.cloudbees.com/hc/en-us/articles/224543927-GitHub-Integration-Webhooks)

1. [Cloud Bees webhook for non multi branch jobs](https://support.cloudbees.com/hc/en-us/articles/115003015691-GitHub-Webhook-Non-Multibranch-Jobs)

[by](http://synved.com/wordpress-social-media-feather/)

![](https://cdn-images-1.medium.com/max/2000/0*ART725fptZh11UFG.png)

*Originally published at [blog.tentamen.eu](https://blog.tentamen.eu/jenkins-and-github-integration-using-webhooks/) on June 23, 2018.*
