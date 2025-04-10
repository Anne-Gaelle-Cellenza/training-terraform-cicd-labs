# Manage Terraform In Azure Repo Git

- [Manage Terraform In Azure Repo Git](#manage-terraform-in-azure-repo-git)
  - [Lab overview](#lab-overview)
  - [Objectives](#objectives)
  - [Instructions](#instructions)
    - [Before you start](#before-you-start)
    - [Exercise 1: Import a Git Repository](#exercise-1-import-a-git-repository)
    - [Exercise 2: Create development branches](#exercise-2-create-development-branches)
      - [Clone Repository and create dev branch](#clone-repository-and-create-dev-branch)
      - [Use shared dev branch](#use-shared-dev-branch)
      - [Update backend configurations and variables](#update-backend-configurations-and-variables)
    - [Exercise 3: Protect dev and main branches](#exercise-3-protect-dev-and-main-branches)
  - [Appendix](#appendix)
    - [Generate a PAT (Personal Access Token)](#generate-a-pat-personal-access-token)

## Lab overview

In this lab, you will learn how to use Source Control for Terraform templates.

## Objectives

After you complete this lab, you will be able to:

-   Create an Azure Git repository
-   Use Source Control for Terraform templates.

## Instructions

### Before you start

- Check your access to the **Azure Subscription** and **Resource Group** provided for this training.
- Check your access to the **Azure DevOps Organization** and **project** provided for this training.
- Using the Azure portal, create a Storage Account with a `tfstates` container. We will use that container as backend for the the tfstate file.


### Exercise 1: Import a Git Repository

We are going to use an existing repository and import it in Azure DevOps.

From Azure DevOps portal, go to the *Repos* blade:

![git_repo](../assets/git_repo.PNG)

Under the Repository dropdown list select Import Repository:

![git_import](../assets/git_import.PNG)

In the Import blade:

- Leave the Repository type to Git
- For the the Clone URL, use **https://github.com/smartinez-cellenza/training-terraform-cicd-base.git**
- Name your repository **terraform-sample**

The import will start and create a new Azure DevOps Git repository.

![rimport_running](../assets/git_import_running.PNG)

The repository is imported and ready to use.

![import_done](../assets/git_imported.PNG)

### Exercise 2: Create development branches

In this exercise, we are going to create two branches:
- `feat/updateconf` to update the Terraform configuration,
- `dev` to gather all DEV developments.  

Note:  
If this exercise is shared between several people, and you all want to share the same `dev` branch, you might 
1. wait for one person to create the `dev` branch, 
2. wait for the branch to be published to remote, 
3. use it next on your local repo (`git checkout dev` after the repository is locally cloned) > See [Use shared dev branch](#use-shared-dev-branch).  

Or you can also create several different (personal) `dev` branches, naming them accordingly (e.g. `dev-myName`).

#### Clone Repository and create dev branch

Get the clone URL of your repository:

![git_clone_url](../assets/git_clone_url.PNG)

Run the following command to clone the repository:

```powershell
mkdir training_terraform_cicd
cd training_terraform_cicd
git clone repository_url_from_previous_step
```

> If this is the first time you use this Azure DevOps Organization, provide authentication information.
> You can for example generate a PAT token and use (See [Generate a PAT (Personal Access Token)](#generate-a-pat-personal-access-token)).

Go to the cloned Repository folder:

```powershell
cd ./terraform-sample/
```

Create a new local branch, named `dev` (or `dev-myName`):

```powershell
git checkout -b dev
```

Push this local branch to Azure DevOps:

```powershell
git push --set-upstream origin dev
```

#### Use shared dev branch

[This concerns the case where a single `dev` branch is created by one person and shared with others.  
 Jump to next section in case you're not concerned.]  

When the `dev` branch has been pushed to remote, it can be locally retrieved by someone else.  
We first fetch remote contents, and next move to the branch:

```powershell
git branch -a  # <-- list all local + remote branches
git fetch      # <-- retrieve remote contents
git branch -a  # <-- dev branch is now visible
git checkout dev  # <-- move to dev branch
```

#### Update backend configurations and variables

Create a new local branch from the `dev` branch, named `feat/updateconf`:

```powershell
# Ensure you are on the dev branch.
git status
git checkout -b feat/updateconf
```

In the configuration folder, update the `backend.hcl` file for **each and every environments**. Update `resource_group_name` and `storage_account_name` to match the Storage Account you created earlier.

> For this lab, we will be using same Storage Account and container for all environments. Only the `tfstate` file name will be different.  
> In real world, you might want to use a dedicated subscription/storage for each environment.

Add the updated files for the next commit:

```powershell
git add .
```

Create a new commit:

```powershell
git commit -m "update backend configuration"
```

In the configuration folder, update the `var.tfvars` file for **each environment**.
- `resource_group_name`: The name of the Resource Group you are using for this training
- `admin_account_login`: We are going to deploy an Azure SQL Database Server. This variable will be used to set the Administrator account login
- `project_name`: The name of the project. It will be used along with the environment variable to build the SQL Server name.

Add the updated files for the next commit:

```powershell
git add .
```

Create a new commit:

```powershell
git commit -m "update tfvars"
```

Push this local branch to Azure DevOps:

```powershell
git push --set-upstream origin feat/updateconf
```

### Exercise 3: Protect dev and main branches

> To set branch policies, you must be a member of the *Project Administrators* security group or have repository-level *Edit policies* permissions.

Go to the project settings -> Repositories.  
Select the *terraform-sample* project.  
Select the *Policies* blade.  
Under *Branch Policies*, select the `main` branch.  

Activate the option **Require a minimum number of reviewers**
- set the *Minimum number of reviewvers* to 1 
- and *Allow requestors to approve their own changes*.  

> No *apply* is needed, settings are directly taken into account.

This will prevent direct commit to the main branch, and only allow Pull Request.  

> This configuration is not suitable for a real world project, but it allows you to complete pull request for this lab.

Apply the same configuration to the **dev** branch.

## Appendix

### Generate a PAT (Personal Access Token)

From Azure DevOps portal, if needed, generate a new Personal Access Token:

![git_pat](../assets/git_pat.PNG)

Click on *New Token*

![git_pat_config](../assets/git_pat_config.PNG)

- Name : training
- Scope : Code -> Read & write

Click on Create and **copy** the generated Token! (will not longer be available after  you close the window)

> This token can be used to clone the repository.
