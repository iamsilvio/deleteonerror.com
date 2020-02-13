---
title: "Install Office365 without Teams and OneDrive"
subtitle: "At least remove the social stuff if you have to use it."
date: 2020-01-18T01:19:51+01:00
draft: true
tags: ["OneDrive", "Skype", "MS Teams", "office365", "Office Deployment Tool"]
categories: [windows, itstuff]
---

Are you sick of getting stuff you never ordered?  
I had to set up a new Windows system and I remembered that the last time I tried to remove *Microsoft Teams*, *Skype for Business* and *OneDrive* from my Windows installation it was really annoying. So this time I had to go in a different direction. Instead of installing the complete package from *office.com*, I decided to install a custom package. Here is a short description how to do an office installation without the desired applications.

## Step 1: Office Deployment Tool

The Office Deployment Tool is intended for administrators to customize the Office installation for use in local networks, but it will also do the trick for us.

- Official Microsoft download page [OfficeDeploymentTool](https://www.microsoft.com/en-us/download/details.aspx?id=49117)

After you have downloaded the executable, run it and select a directory to extract the contents to.  
You should see some *configuration-Office....xml* files and a setup.exe and thats all for Step 1.

## Step 2: Configuration

Next, we need to configure our office installation according to our needs.  
in most cases the *configuration-Office365-x64.xml* will be the right one, open the file in a text editor.  
My configuration file looks like this:

```xml
<Configuration>
  <Add OfficeClientEdition="64" Channel="Monthly" SourcePath="R:\preload" AllowCdnFallback="True">
    <Product ID="O365ProPlusRetail">
      <Language ID="en-us" />
      <ExcludeApp ID="Teams" />
      <ExcludeApp ID="Access" />
      <ExcludeApp ID="Groove" />
      <ExcludeApp ID="Lync" />
      <ExcludeApp ID="OneNote" />
      <ExcludeApp ID="OneDrive" />
      <ExcludeApp ID="Publisher" />
    </Product>
    <Product ID="LanguagePack">
      <Language ID="de-de" />
    </Product>
  </Add>
  <Updates Enabled="TRUE" Channel="Monthly" />
  <Display Level="None" AcceptEULA="TRUE" />
  <Property Name="AUTOACTIVATE" Value="1" />
  <Logging Level="Off" Path="%temp%" />
</Configuration>
```

Additional resources to the configuration file format can be found at [docs.microsoft.com](https://docs.microsoft.com/en-us/deployoffice/configuration-options-for-the-office-2016-deployment-tool)  
The List of all Apps can also be found at the [docs](https://docs.microsoft.com/en-us/deployoffice/configuration-options-for-the-office-2016-deployment-tool#id-attribute-part-of-excludeapp-element)  

Thats all for Step 2, after you have customized your installation we jump to the installation.

## Step 3: Download and Installation

Now there are only two things left, download the installation package and start the installation.  
  
Open a command window or powershell console and navigate to the folder containing the setup.exe.
The command `.\setup.exe /download configuration-Office365-x64.xml` will download all needed files to the filesystem Path which you have defined as **SourcePath** in your XML configuration.  
This is gonna take a while, so why don't you just sit back, relax and get yourself a cup of coffee.
  
You have done it right away, when the download is finished, you only have to execute the following command to start the installation.
`.\setup.exe /configure configuration-Office365-x64.xml`.  
**Done**

## Finaly

You can keep the installation files if you think you will need them again, or you can simply delete both folders.
