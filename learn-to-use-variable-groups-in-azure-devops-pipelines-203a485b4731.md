Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m20[39m }

# Learn to Use Variable Groups in Azure DevOps Pipelines

Learn to Use Variable Groups in Azure DevOps Pipelines

A few months ago, I set up an AWS development environment in one region with Terraform and Chef. I realized I needed to refactor my code to handle multiple regions aka multi-region and AWS accounts. I didn’t want to duplicate code for the various regions and environments. The solution would become a nightmare to maintain and have a horrible code smell. Ew.

I was new to Azure DevOps for CI/CD pipelines and I wanted to find a way to apply different sets of variables to the same code base, so I poked around and found Variable Groups. ‘This sounds promising!’ I thought and started digging through Microsoft docs to learn more. So here’s the scoop on Variable Groups.

* You can share variable groups between pipelines.

* You can “link’“ variable groups to specific stages in a pipeline, build, or release.

## Variable Groups

To create a Variable Group, follow the steps below:

1. In the left panel, click and expand *Pipelines*.

1. Under *Pipelines*, click *Library*.

1. Click the *Variable group* button.

1. Enter a name for the variable group in the *Variable Group Name* field.

1. Use the *Description* field to enter information about the variable group.

1. Click the *Add *button to create a new variable for the group.

1. Fill in the variable *Name* and *Value*.

![](https://cdn-images-1.medium.com/max/2000/0*Ill0TNodr-4UMg07.png)

An example variable group for AWS us-east-1.

For this tutorial, we’ll link our Variable Groups to an Azure DevOps Build and Release Pipeline. I’ll show two methods:

1. As code with azure-pipeline.yml file in the Build Pipeline.

1. With the Azure DevOps GUI in the Release Pipeline.

## Build Pipeline

Before we can link the variable group to our build we need to set up a repository. The repository can be set up in Azure DevOps or GitHub. For this tutorial, I used Azure DevOps. I’ve also posted the code on [GitHub](https://github.com/kellimohr/DevOpsDiva/tree/master/azure_variable_groups) but it’s pretty basic output to demonstrate how the Variable Groups function.

Lets set up the build with the steps below:

1. In the left panel, expand Pipelines.

1. Click on the New pipeline button.

1. Associate the pipeline to a repo in GitHub or Azure DevOps.

1. Click Run.

As we can see, our first build succeeds as expected, but none of the variables were echoed during the build.

![](https://cdn-images-1.medium.com/max/2000/0*eRUbdZyMWk3K5StB.png)

![](https://cdn-images-1.medium.com/max/2000/0*G3mNl-Qyhn2jaaX-.png)

Next, lets associate the variable groups the build. Since the build uses the azure-pipelines.yml file , we’ll update the code to use the variable groups. For more information on Azure Pipelines, check out Microsoft’s [YAML schema reference](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema).

In your editor, open up azure-pipelines.yml and add the code snippet below to the beginning of the file.
> # variables:
> # - group: dev_us-west-2

Save and commit the file. The build should execute and output the variables stored in the variable group dev_us-west-2 (see below). Now try it with dev_us-east-1.

![](https://cdn-images-1.medium.com/max/2000/0*B_wnpDqMDoXT0VrR.png)

Let’s take our variable groups a step further and create two jobs: “Build us-east-1” and “Build us-west-2”

For this exercise, we’ll match the appropriate variable group to the it’s job. Open up the azure-pipelines.yml file in your favorite editor and replace the text provided in this [GitHub branch](https://github.com/kellimohr/DevOpsDiva/blob/variable_groups_jobs/azure_variable_groups/azure-pipelines.yml). Or use the code below.

![](https://cdn-images-1.medium.com/max/2000/0*ZS1jyLIZXEmROxmP.png)

After you’ve saved your changes, commit and push the code to run the build. Now you see two separate jobs, one for “build_Virgina” and another for “build_Oregon” each of our regions.

![](https://cdn-images-1.medium.com/max/2000/0*i4E43m-Z6pmCaBSJ.png)

Open each job and click on the “Show Variable Values” step. The values echoed for each Variable Group correspond to the assigned job in the code block above. See the images below.

![](https://cdn-images-1.medium.com/max/2000/0*XGG-TMXVyTVa11W7.png)

![](https://cdn-images-1.medium.com/max/2000/0*_s3q0PMnbGtz551M.png)

Now that we learned how to associate Variable Groups to build pipelines through code, let’s move on to Releases!

## Release Pipeline

For this section of the tutorial, we will create a Release Pipeline for both of our regions, us-east-1 and us-west-2. Use the steps below to create a new Release in Azure DevOps.

1. In the left hand panel, click *Pipelines*.

1. Click* Releases*.

1. Click the *New pipeline* button.

1. A new pane will open for *Stage 1,* prompting you to *Select a template* Or start with an *Empty job*.

1. Click the *Empty job* link.

1. A new pane opens for the stage. Change the *Stage name* to Release us-east-1.

1. Use the *x* in the upper right corner to close the pane.

![](https://cdn-images-1.medium.com/max/2000/0*NF7OOb8Qr-HScipV.png)

Next, we will link the Variable Group dev_us-east-1 to the Release us-east-1 stage. To do this, follow the steps below:

1. Click the *Variables* menu.

1. Select *Variable group*s in the middle panel.

1. Click the *Link variable group* button.

1. Select *Dev_us-east-1*.

1. Under the Variable group scope, select the *Stages* radio button.

1. In the drop-down box, select *Release us-east-1*.

1. Click the *Link* button.

1. Click the *Pipeline* to return to the *Stages*.

![](https://cdn-images-1.medium.com/max/2000/0*codCrJgSMX95Sf0h.png)

Now we need to create a new task for our release. To keep it simple for this tutorial, we’ll write a script to echo the values of our pipeline variables to validate our Variable Group has been linked.

1. Click the *1 job, 0 task* link under the Release us-east-1 stage.

1. Click the blue* +* on the *Agent job* section.

1. A new pane opens, search for or select *Command Line*.

1. Under the *Agent job* section, Click *Command Line Script*.

1. Update the *Display name* to *Echo Variable Group Values*.

1. Add the snippet below to the* Script *text box.

echo %REGION%

echo %STATE_BUCKET%

echo %INSTANCE_SIZE%

1. Click *Save*.

1. Click *Release*.

1. Select *Create a Release*.

1. A new pane opens, click the *Release* button.

![](https://cdn-images-1.medium.com/max/2000/0*a_BYGkKUgz7zukAB.png)

After the release begins, a banner displays on screen. Click the *Release-1 *link provided to open the details and see the output. Use the steps below.

1. After the release completes, click the *Release us-east-1 *stage box.

1. Click the* Echo Variable Group Values* in the *Agent job* section.

The variable values for dev_us-east-1 have been exported successfully for this stage of the release.

![](https://cdn-images-1.medium.com/max/2000/0*o_xu0WFbZxbHBYQX.png)

![](https://cdn-images-1.medium.com/max/2000/0*DiyXO351vFmXSVqi.png)

Finally, we will create a new stage for *Release us-west-2* and associate this stage with the dev_us-west-2 Variable Group. Go back to the *Pipelines* and *Releases sections* to edit the existing release.

1. Hover over the *Release us-east-1* box, click the Clone button (looks like two papers stacked).

1. A copy of the release appears, click the box to edit.

1. Update the *Stage nam*e to *Release us-west-2*.

1. Click the *Lightening Bolt* icon.

1. Change the *Triggers *to *After Release*.

1. Click the *Variables* menu.

1. Click the *Variable groups* section.

1. Click the *Link variable group* button.

1. Select *dev_us-west-2*.

1. Under the *Variable group scope*, click the* Stages* radio button.

1. This time select *Release us-west-2*.

1. Click the *Link* button.

1. Click the *Save* button in the upper right.

![](https://cdn-images-1.medium.com/max/2000/0*iAKardfTOtt2wegP.png)

Now we have separate stages to release to our two AWS regions in us-east-1 and us-west-2. As you can see, Azure DevOps Variable Groups gives us the opportunity to apply one code base to many different environments and the ability to reduce repeated code.

![](https://cdn-images-1.medium.com/max/2000/1*ZZt5P5_1FPd0cOyTO_f4hw.gif)
