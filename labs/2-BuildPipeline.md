# Create a build pipeline

- [Create a build pipeline](#create-a-build-pipeline)
  - [Lab overview](#lab-overview)
  - [Objectives](#objectives)
  - [Instructions](#instructions)
    - [Before you start](#before-you-start)
    - [Exercise 1: Create a PR pipeline](#exercise-1-create-a-pr-pipeline)
    - [Exercise 2: Add this pipeline to the policies on main branch](#exercise-2-add-this-pipeline-to-the-policies-on-main-branch)
    - [Exercise 3: Create a pull request from dev to main](#exercise-3-create-a-pull-request-from-dev-to-main)
    - [Exercise 4: Produce an artefact when changes are merged on the main branch](#exercise-4-produce-an-artefact-when-changes-are-merged-on-the-main-branch)
    - [Exercise 5: Create a pull request from dev to main](#exercise-5-create-a-pull-request-from-dev-to-main)
    - [Exercise 6: Rename the build pipeline](#exercise-6-rename-the-build-pipeline)

## Lab overview

In this lab, you will learn how to use Build pipeline.

## Objectives

After you complete this lab, you will be able to:

-   Create a build pipeline using yaml,
-   Trigger this pipeline when a Pull Request is created and set it mandatory.

## Instructions

### Before you start

- Check your access to the Azure Subscription and Resource Group provided for this training.
- Check your access to the Azure DevOps Organization and project provided for this training.
- Project has branch configuration according to the lab *1-Manage Terraform In Azure Repo Git*.

### Exercise 1: Create a PR pipeline

In this exercise, we will create a validation pipeline that will check the formating style of Terraform templates.

Select your *terraform-sample* in Azure DevOps portal.  
Select the *dev* branch.  
Create a new folder, and a new file inside it:
- New folder name: pipelines
- New file name: PR.yml

![create_folder](../assets/build_create_folder.PNG)

Copy the following code in the editor:

```yaml
trigger: none

jobs:
- job: Linter
  displayName: Linter
  pool:
    vmImage: ubuntu-20.04
  steps:
  - checkout: self
  - task: PowerShell@2
    inputs:
      targetType: 'inline'
      script: |
        cd ./src/terraform
        terraform fmt -recursive -check -diff
```

Commit this file.  

> Notice the different sections in this yaml file.

Go to the Pipelines blade in Azure DevOps and create a new pipeline:

![new-pipeline](../assets/build_new_pipeline.PNG)

For *Where is your source code* step, select **Azure Repo Git**.  
For *Select a repository* step, select **terraform-sample**.  
For *Configure your pipeline* step, select **Existing Azure Pipelines YAML file**.  
For *Select an existing YAML file*
- select the `dev` branch
- fill the path: **/pipelines/PR.yml**

Click on *Run* to execute the pipeline.

> Check the pipeline execution.  
> This pipeline uses `terraform fmt` to lint terraform source code. If you have errors, fix them and run the pipeline again.

Select the pipeline in the pipeline blade, and rename it to `PR`.

![rename](../assets/build_rename.PNG)

### Exercise 2: Add this pipeline to the policies on main branch

Go to the project settings -> Repositories

Select the *terraform-sample* project.  
Select the *Policies* blade.  
In the *Branch Policies*, select the `main` branch.  
Add a new *Build validation* rule:

![branch_policy](../assets/build_branch_policy.PNG)

Leave the default options.

### Exercise 3: Create a pull request from dev to main

In the Azure Repo blade, select the *terraform-sample* repo.  
In the repository blade sub-menu, select *Pull Requests*.  
Create a new Pull request from `dev` to `main`.  
Enter values for *Title* and *Description*.  

> Notice that the build validation is triggered.  
> Notice also the *At least 1 reviewer must approve* note: you cannot complete the Pull Request until someone has validated it.  

Approve and complete the Pull Request.  

### Exercise 4: Produce an artefact when changes are merged on the main branch

In this exercise we will add another pipeline definition to produce an artefact when new code is merged on the main branch.  

Select your *terraform-sample* in Azure DevOps portal.  
Select the `dev` branch.  
Create a new file under the *pipelines* folder:
- New file name: build.yml

Copy the following code in the editor:

```yaml
trigger:
  branches:
    include:
    - main

jobs:
- job: Artifact
  displayName: Upload artifact
  pool:
    vmImage: ubuntu-20.04
  steps:
  - checkout: self
  - task: PowerShell@2
    inputs:
      targetType: 'inline'
      script: |
        Copy-Item -Path ./src -Destination $(Build.ArtifactStagingDirectory)/terraform -Recurse
        Copy-Item -Path ./configuration -Destination $(Build.ArtifactStagingDirectory)/terraform -Recurse
  - task: PublishBuildArtifacts@1
    inputs:
      PathtoPublish: '$(Build.ArtifactStagingDirectory)/terraform'
      ArtifactName: 'terraform'
      publishLocation: 'Container'
```
Commit this file.  

> This pipeline creates an artefact containing both *src* and *configuration* folders.  
> These files are required to perform a deployment on an environment.

Go to the *Pipelines* blade in Azure DevOps and create a new pipeline:

![new_pipeline](../assets/build_new_pipeline.PNG)

For *Where is your source code* step, select **Azure Repo Git**.  
For *Select a repository* step, select **terraform-sample**.  
For *Configure your pipeline* step, select **Existing Azure Pipelines YAML file**.  
For *Select an existing YAML file*
- select the `dev` branch
- fill the path: **/pipelines/build.yml**

Click on *Run* to execute the pipeline.  
In the pipeline blade, ensure the pipeline has run and artifact has been created:

![artifacts](../assets/build_artefacts.PNG)

> Notice the content of the produced artifact:

![artifact_content](../assets/build_artefacts_content.PNG)

### Exercise 5: Create a pull request from dev to main

In the Azure Repo blade, select the *terraform-sample* repo.  
In the repository blade sub-menu, select *Pull Requests*.  
Create a new Pull request from `dev` to `main`.  
Enter values for *Title* and *Description*.  

> Notice that the two triggered builds
> - the PR build validation 
> - and the publish artefact build.

### Exercise 6: Rename the build pipeline

We will later use this pipeline as source for another one.  
To clearly identity this resource pipeline, we must name it.  
Select the build pipeline in the pipeline blade, and rename it to `build`:

![rename](../assets/build_rename.PNG)

