# terraform-azurerm-securestorage-module
In this hands-on lab, you'll create a Terraform module, publish the module to a private registry in Terraform Cloud, and create a configuration that leverages the module to deploy a secure storage account.



Create a Private Module Registry in Terraform Cloud for Azure
Introduction

In this hands-on lab, you'll create a Terraform module, publish the module to a private registry in Terraform Cloud, and create a configuration that leverages the module to deploy a secure storage account.
Solution

To complete this lab, you will need to use a remote desktop client.

        Windows: Microsoft Remote Desktop on Windows 10
        MacOS: Microsoft Remote Desktop
        Linux: Remmina

To use the Remote Desktop client:

    In the Azure portal, click Virtual machines in the left-hand menu.
    Click the listed lab-vm virtual machine.
    Click Connect.
    Click Download RDP File.
    Open the file in your Remote Desktop app.
    Edit the desktop to:
        Make sure you are not in an admin session.
        Make sure the remote session does not launch in full-screen mode.
        Set the resolution to something that will give you room to maneuver in both the remote and local desktops (e.g., 1024 x 768).
    Log in to the virtual machine using the following credentials:
        User Name: Use the user name provided by the lab credentials.
        Password: Use the password provided by the lab credentials.

    Note: If you're using a Mac, instead of downloading the RDP file, you will need to copy the public IP address of the virtual machine in the Azure Portal and use that to access the virtual machine via your RDP client.

To complete this lab, you will also need:

    Your own free Terraform Cloud account, you can sign up at Terraform Cloud.
    Your own free GitHub account, you can sign up to join GitHub.

Create a GitHub Repository for Your Module

    Once you've logged in to the virtual machine, open a new browser window.
    Log in to the Azure portal using the credentials provided on the lab page. Be sure to use an incognito or private browser window to ensure you're using the lab account rather than your own.
    In a new browser window or tab, log in to your free account in Terraform Cloud.
    In a new browser window or tab, log in to your GitHub account.
    In GitHub, click the New button near the top left of the page next to Top Repositories.
    On the Create a new repository page, set the following parameters:

    Owner: Select yourself.
    Repository name: Enter terraform-azurerm-securestorage.
    Public: Select the radio button next to this option.
    Add a README file: Check the checkbox next to this option.
    Add gitignore: Select terraform from the dropdown menu.

    Click the Create repository button.

Author Your Terraform Module
Clone the GitHub Repository

    Click the <> Code button.
    Copy the repository URL.
    Open Visual Studio Code by clicking the blue Visual Studio Code icon in the Windows taskbar at the bottom of the screen.
    Press Ctrl + Shift + P to open the Control Palette (or Cmd + Shift + P on a Mac).
    Enter git clone in the Command Palette.
    Select Git Clone from the search results.
    Paste the GitHub repository URL.
    Click the Documents folder.
    Click the Select as Repository Destination button.
    When asked if you would like to open the cloned repository, click the Open button.
    Click the Yes, I trust the authors button.

Create Terraform Files for the Module

    In the left pane, right-click and select New File to create a new file.

    Enter main.tf for the file name.

    In the left pane, right-click and select New File to create another new file.

    Enter variables.tf for the file name.

    In the left pane, right-click and select New File to create a third new file.

    Enter outputs.tf for the file name.

    Right-click on variables.tf and select Open to the Side.

    Right-click on output.tf and select Open to the Side.

    Open main.tf.

    In main.tf, create the Terraform block:

    terraform {
      required_version = ">= 1.3.0"

      required_providers {
        azurerm = {
          source  = "hashicorp/azurerm"
          version = ">= 3.43.0"
        }
      }
    }

    Save the file by pressing Ctrl + S (or Cmd + S on a Mac).

    Select Terminal from the Visual Studio Code taskbar.

    From the menu, select New Terminal.

    In the terminal, initialize the working directory in Terraform:

    terraform init

    In the main.tf file, create a local variable that can used to assign tags to resources in the module:

    locals {
      tags = {
        "Environment" = var.environment
      }
    }

    Create a storage account resource block:

    resource "azurerm_storage_account" "securestorage" {
        resource_group_name = var.resource_group_name
        location = var.location
        name = var.storage_account_name
        account_tier = "Standard"
    #Explain that modules should be opinonated
    #Explain conditional statements
        account_replication_type = var.environment == "Production" ? "GRS" : "LRS"
        public_network_access_enabled = false

       tags = local.tags
    }

    Save the file by pressing Ctrl + S (or Cmd + S on a Mac).

    In the variables.tf file to the right, create the variable for the resource group name:

    variable "resource_group_name" {
        type = string
        description = "The resource group name"
    }

    variable "location" {
        type = string
        description = "The resource location"
    }

    variable "storage_account_name" {
        type = string
        description = "The storage account name"
    }

    variable "environment" {
        type = string
        description = "The environment either Production or Development"
    }

    Save the file by pressing Ctrl + S (or Cmd + S on a Mac).

    In the outputs.tf file, define the output:

    output "storage_account_id" {
      value       = azurerm_storage_account.securestorage.id
      description = "The storage account name"
    }

    Save the file by pressing Ctrl + S (or Cmd + S on a Mac).

Push the New Terraform Files to GitHub

    In the terminal, validate the configuration in a directory:

    terraform validate

    Config your Git user name:

    git config user.name "<your name here>"

    Config your Git password:

    git config user.email "<your email address here>"

    In the left bar on Visual Studio Code, click the Source Control icon.

    Stage the main.tf file by pressing the + icon next to the file name.

    Stage the outputs.tf file by pressing the + icon next to the file name.

    Stage the variables.tf file by pressing the + icon next to the file name.

    Stage the .terraform.lock.hcl dependency lock file by pressing the + icon next to the file name.

    Above the Commit button, enter a message to describe the commit, such as version 1.0.0.

    Click the Commit button.

    Click the Sync Changes button.

    When told that this action will pull and push commands from and to "origin/main", click the OK button.

    Click the Sign in with your browser option.

    Go to the browser window or tab that has just opened and click the Continue button.

    Return to Visual Studio Code and verify that our changes have been successfully pushed by checking the numbers in the bottom left of Visual Studio Code.

Publish Your Module to Your Private Registry

    Go to the browser window or tab with GitHub open.
    Select tags.
    Click the Create a new release button.
    Click the Choose a tag button.
    Enter 1.0.0 as the tag's name.
    Select + Create new tag 1.0.0 on publish.
    Enter the name as Version 1.0.0.
    In the description, enter First release.
    Click the Publish release button.
    Go to the browser window or tab with Terraform Cloud open.
    In the left-hand navigation menu, select Registry.
    Select Publish a module.
    Under Connect to a version control provider, select GitHub. (If you have not configured GitHub as a version control provider, you can find instructions on how to do so in this tutorial on connecting GitHub to Terraform).
    Select the repository you created. If it isn't showing up in the list, you can also enter in the name (which should be <Your account name>/terraform-azurerm=securestorage) for your repository under Can't see your repository? and click the arrow on the side.
    Click the Publish module button. This may take a few minutes to complete.

Author and Apply a Configuration Using the Module
Create the main.tf File

    Go back to Visual Studio Code.

    Select File in the taskbar.

    From the menu, select Open Folder.

    Select New folder.

    Name the folder storage.

    Click the Select Folder button.

    Click the Yes, I trust the authors button.

    In the left pane, right-click and select New File.

    Name the new file main.tf.

    In the main.tf file, create the Terraform block:

    terraform {
      required_providers {
        azurerm = {
          source  = "hashicorp/azurerm"
          version = ">= 3.43.0"
        }
      }
    }

    Add a provider block:

    provider "azurerm" {
        features {}
        skip_provider_registration = true
    }

Create a Credentials File

    Go back to the browser window or tab with Terraform Cloud open.
    Copy the credentials block on the right side of the screen.
    Return to Visual Studio Code.
    Select File from the Visual Studio Code taskbar.
    From the menu, select New File.
    Select Text File.
    Paste in the credentials block in the new file.
    Go back to the browser window or tab with Terraform Cloud open.
    In the left navigation pane, click the user profile icon in the top right corner of the pane.
    Select User Settings.
    In the left-hand navigation menu, select Tokens.
    Click the Create an API token button.
    When prompted for a description, enter User profile.
    Click the Create API token button.
    Copy the generated token.
    Return to Visual Studio Code and paste the token into the credentials block.
    Select File in the Visual Studio Code taskbar.
    From the menu, select Save As.
    In File name, enter %APPDATA%, which will navigate you to the AppData directory.
    Once in the AppData directory, enter "terraform.rc" in File name.
    Click the Save button.
    Close the the terraform.rc file.

Finish Authoring the Configuration

    In the main.tf file, add the block for the existing resource group:

    resource "azurerm_resource_group" "rg" {
        name = "" 
        location = ""
    }

    Go back to the browser window or tab with the Azure portal open.

    In the left-hand navigation menu, select Properties under Settings.

    Copy the name under Name to your clipboard.

    Go back to Visual Studio Code and paste the name between the two double quotes after name =.

    Go back to the browser window or tab with the Azure portal open.

    Copy the ID under Location ID to your clipboard.

    Go back to Visual Studio Code and paste the ID between the two double quotes after location =.

    Save the file by pressing Ctrl + S (or Cmd + S on a Mac).

    Go back to the browser window or tab with Terraform Cloud open.

    In the left-hand navigation menu, click the Home icon in the top right of the menu.

    In the left-hand navigation menu, select Registry.

    Select the securestorage module.

    In the right-hand pane, copy the configuration details under Copy configuration details.

    Go back to Visual Studio Code and paste in the configuration details in main.tf.

    Save the file by pressing Ctrl + S (or Cmd + S on a Mac).

    In the Visual Studio Code taskbar, select Terminal.

    From the menu, select New Terminal.

    In the terminal, download the azurerm provider and the securestorage module:

    terraform init

Add the Required Variables

    Add the resource group name, location, environment, and storage account name to the module block in the main.tf file so that it looks like this:

    module "securestorage" {
      source  = "app.terraform.io/ps-wayne-hoggett/securestorage/azurerm"
      version = "1.0.0"
      resource_group_name = azurerm_resource_group.rg.name
      location = azurerm_resource_group.rg.location
      environment = "Production"
      storage_account_name = "<globally unique storage account name>"
    }

    Make sure to use a globally unique name for the storage_account_name, even by just changing the string of numbers at the end.

    Save the file by pressing Ctrl + S (or Cmd + S on a Mac).

    In the terminal, make sure the file is valid:

    terraform validate

    Log in using the Azure ClI:

    azure login

    This will open a browser to authenticate our Azure account.

    Select Cloud Student. You can close the tab afterward.

    Go to the browser window or tab with the Azure portal open.

    Copy the ID under Resource ID.

    Go back to Visual Studio Code.

    Import the existing resource group and paste in the resource ID at the end:

    terraform azurerm_resource_group.rg <paste resource ID here>

    Deploy the resource group and apply a Terraform plan:

    terraform apply

    When prompted to enter a value, enter yes. The deployment may take a few minutes.

    Go back to the browser window or tab with the Azure portal open.

    In the breadcrumb trail on top of the page, click the resource group name to navigate back to the resource group.

    Select Refresh in the Azure portal. You should see the storage account listed in the resources of your resource group.

    Select the storage account. You should see it has the Environment: Production tag and is using geo-redundant storage.

Conclusion

Congratulations — you've completed this hands-on lab!