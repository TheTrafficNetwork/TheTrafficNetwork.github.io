---
title: "IOS v12/15 update using archive tar /xtract" #Auto-Populated Title of the Post
date: 2022-01-16T20:13:48-06:00 #Auto-Populated Date Field
draft: false #Status of the Post
author: "TheTrafficNetwork" #Author Name
description: "Updating Cisco IOS v12 and v15 using archive tar /xtract" #Description/Used in SEO
summary: "Updating Cisco IOS v12 and v15 using archive tar /xtract" #Used on Archive Pages and in RSS
images: ["/images/ios-updates/gui_dashboard.png","/images/ios-updates/tar_xtract.png","/images/ios-updates/ios_download.png"] #images used for social media preview. comma separate each image path enclosed in double quotes
#thumbnail: "/images/ios-updates/new_show_version.png" #featured-image of the page. i will recommend using same image for both preview and thumbnail
tags: ["IOS","Cisco","Networking"] #comma separated tags enclosed in double quotes. also used for SEO.
categories: [] #comma separated categories enclosed in double quotes.
#series: [] #A taxonomy used to list "See Also" Section in Opengraph Templates
slug: "ios-v12-v15-upgrade" #Similar to WordPress's Slug (the end part of the url)
disableComments: true #Set to 'true' if you need to disable comments for any post
---

## Do you need to be able to utilize the GUI interface for Cisco devices?

Maybe not. You might be the master of the CLI or automation, but the GUI could still provide utility to people that can help make your life easier. For instance, we have traffic signal technicians that deploy various devices at our signalized intersections, but they don't have any experience programming a switch, or have access to the cli. What the GUI does allow them access to is the ability to see if there is a port error, or a mismatched vlan assingment, or even what the port description for where they need to deploy devices to while still in the shop before they go to the field and start troubleshooting a problem that they might not be able to. The GUI saves time for us, and money for the taxpayers.

So how do we perform IOS upgrades with the GUI? When you navigate through [Cisco's software download site](https://software.cisco.com/download/home) to the model of your device, you will find that there is a .bin file and a .tar file. The .bin is your standard IOS upgrade. It can be downloaded to a workstation or server and then through the use of a tftp application, get transferred over to the switch. With a simple change of the boot location and a reload, Ta-Da, a new IOS version is running on your switch. The GUI application file is almost as simple, it just takes more time and utilizes a different file transfer command. Let's dive in!

## Here are the steps we will perform:

1. Check the current version
   - ```Switch#show version```
2. Download the new IOS file
   - <https://software.cisco.com/download/home>
3. Check the device to see if we have enough room for the new image
   - ```Switch#dir flash:```
4. Transfer and extract the .tar file to the device's flash:
   - ```Switch#archive tar /xtract {remote-host}{file}.tar flash:```
5. Change the boot location
   - ```Switch(config)#boot system flash:/{folder}{file}.bin```
6. Reload the device
   - ```Switch#reload```
7. Verify the new version
   - ```Switch#show version```
8. Delete the old files
   - ```Switch#delete /recursive /force flash:/{old-folder}```

Let's get into the details.

## CHECK THE CURRENT VERSION

To see the version of software we are currently running, a ```show version``` command is used:

**Switch#show version**{{< figure src="/images/ios-updates/old_show_version.png" caption="The current version is 15.0(2)SE5" >}}

## DOWNLOAD THE NEW IOS FILE

Looks like we are using an older version so let's download the updated software from [Cisco's software download site](https://software.cisco.com/download/home) and navigating to the device in question: a Catalyst 2960C-12PC-L. Like mentioned before, the .tar file is what we are looking for in the download pages. This file includes the IOS upgrade as well as the web based manager. {{< figure src="/images/ios-updates/ios_download.png" >}}

## MAKE SURE THERE IS ENOUGH ROOM IN THE FLASH FOR THE NEW IMAGE

After we have the download and see how much space we have available in our device's flash with the command: **```dir flash```** and compare the free space to the size of the .tar file. If there is not enough room, you can remove files from the flash: with the **```delete flash:/{file}```** command.

**Switch#dir flash**

 ***Disclaimer: Before you delete any files on your flash, make sure you know exactly what they do and if they are unused. Files that are safe to delete are typically older IOS versions that you are not currently booted to.***

## TRANSFER AND EXTRACT THE .TAR FILE TO THE DEVICE'S FLASH

It looks like there is enough room to get the image on the device so do we just copy it over .bin style? How would we set the boot command to the IOS image then? We must perform an extraction on the .tar file to get all the files out of the .tar and installed into the flash. The command needed to accomplish this is: **```archive tar /xtract {source} {destination}```**. Since this is being performed on an isolated lab, I will use a tftp server on my workstation as I don't anything more secure and we will transfer the files to the **```flash:```** of the device.

**Switch#archive tar /xtract tftp://10.10.10.120/c2960c405-universalk9-tar.152-7.E3.tar flash:** {{< figure src="/images/ios-updates/tar_xtract.png" caption="The waiting game begins...">}}

## CHANGE THE BOOT LOCATION

The extraction process creates a new folder in the flash that uses the name of the .tar file. Within that folder is a .bin file with the same name. For the device to boot into the new IOS version, we need to issue a new boot command in configuration to point at the newly created bin file.

**Switch(config)#boot system flash:/c2960c405-universalk9-tar.152-7.E3/c2960c405-universalk9-tar.152-7.E3.bin**

## RELOAD THE DEVICE

The next time the switch boots up, it will load the new ios. Let's force that now with a **```reload```** command:

**Switch#reload**{{< figure src="/images/ios-updates/flash_loading.png" caption="You can see the updated version here as well" >}}

## VERIFY THE NEW VERSION

Like in the first step, we will use the **```show version```** command.{{< figure src="/images/ios-updates/new_show_version.png" caption="Updated to version 15.2(7)E3" >}}

## DELETE THE OLD FILES

If your device did not have the GUI installed on the previous version, deleting the old file should be as easy as using the **```delete flash:/{file}```** command. However, if there was a GUI interface installed, there is an entire folder structure that you need to remove and a basic delete file command will not work. There are some command options that you must apply so that a folder is forceably removed. These are **```/recursive```** and **```/force```**.

**Switch#delete /recursive /force flash:/c2960c405-universalk9-mz.150-2.SE5**{{< figure src="/images/ios-updates/delete_old_folder.png" >}}

## ACCESS THE WEB MANAGEMENT IN BROWSER

Now in a web browser, navigate to your management IP address to view the GUI. To get in, you will need to create a login credentials and set ```http authentication {aaa|enable|local}``` under the switch config accordingly.{{< figure src="/images/ios-updates/gui_dashboard.png" >}}

There you have it. A switch with an updated IOS complete with Web Based Manager. Enjoy!
