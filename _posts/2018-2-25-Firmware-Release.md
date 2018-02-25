---
layout: post
title: Embedded software release procedure
published: true
---

One of my first job assignments as a Junior Embedded software developer was supporting a source code of a simple project. It was a pretty simple thing — a small board with an 8-bit processor, a couple of buttons, a couple of LEDs and solid state relay. LEDs were used to show the actual state of the board. There was an error condition where both LEDs were blinking quickly. 

One day, a customer contacted me, saying he doesn’t like how the LEDs were blinking during an error condition. They were blinking “way too fast” and must be speed down. Ok, pretty simple. Half an hour later, I sent the customer an email with a new hex file attached. “This is still too fast, let's slow it down a little more.” Ok, another email with new firmware build. “Still too fast!” Ok, another email. Now, I changed the period to 1Hz — it’s really slow. “No, they are still blinking as fast as before, I ever can notice the difference.”

Well, what’s going on here? I got the customer on the phone and finally found the issue. The problem was very simple: they were uploading a wrong file to the board. They were saving the attachment somewhere on the disc, then got the firmware update tool and selected an original, old firmware file delivered months ago! They were downloading the same firmware file repeatedly. This was why they saw the LED blink always at the same speed: it was the same firmware all the time. 

# Release matters
I got a lecture from this simple experience. Being a software developer is much more than just writing code. Documentation matters. Bug tracking matters. And management of software release matters.

Almost every IDE allows you to choose between “Debug” and “Release” build. And almost in every IDE, the “Release” build is just a small .hex or .bin file in some subfolder of your project directory. Each time the code is recompiled, the file is being overwritten with the most recent version.

What’s actually wrong with that? While you are working solo on the project, this is not so bad. The problems start when it comes to releasing or distributing the firmware. And I’m not necessarily talking about going production. A release happens when a hardware engineer from your team is requesting the latest firmware to test a newly assembled board. It could be an app developer asking for updated firmware with the new Bluetooth API feature. It even could be a customer with access to an early prototype version, asking for some features or small changes.

In any case like these, one sends the hex file over email, attach to a Slack channel, or whatever way the team uses to communicate. Later, your team will refer to that file like “the latest version” or “the file you sent on some-date-long-time-ago” or maybe invent some fancy name “test version” or “updated API version.” As new versions are released, things just became worse. Probably, after some time-wasting miscommunication episodes, the team comes with an idea to start versioning firmware releases.

## firmware_release_v1.6_final_3_fixed_8.hex is not a good idea

It’s very likely all of us meet at some point in our professional lives a file name convention like this … if we can call this convention at all. Yes, any versioning convention is better than just sending “firmware.hex” files out there. But naming the file in a chaotically random manner is definitely not a solution.

# A simple solution

Since there is no solution out of the box on most IDEs, why not implement it? At the end of the day, we are software developers, and this is what we do; we write software to solve problems.

Said this, a small, simple batch script is all we need to start figuring this out. Here is a script example for an Atmel Studio 7 project:

	[script goes here]

This simple script receives 3 input parameters:
* Path to the project directory
* Build configuration name (i.e., debug, release, etc.)
* Assembly file name (i.e., the name of the .hex file)

Let's store the script in file “deploy.cmd” in a “deploy” folder under project structure. In the same folder, create a “build.txt” file. The only content of this file is a current build number.

What the script does is pretty simple: 
* Read the build number from the build.txt text file
* Make a copy of the hex file and rename it to include the build number in the name 
* Increase the build number by one and save the new value to build.txt file

Simple, right? Now, we need to configure Atmel Studio to run this script after each build. This can be done from Project -> Properties -> Build Events. Simply setup this command in “Post-build event command line” text box:

	"$(MSBuildProjectDirectory)\deploy\deploy.cmd" "$(MSBuildProjectDirectory)" $(Configuration) $(AssemblyName)

![firmware-release-atmel-studio-screenshot.png]({{site.baseurl}}/images/posts/firmware-release-atmel-studio-screenshot.png)
Atmel Studio Build Events Configuration and “hex” folder

## Going one step further
Once we have the script tested and working, the next step is to add the build number to our source code. It’s really easy to do; we just need another simple script.

	[script goes here] 

This super simple script read the build number from the same txt file and creates the “build.h” file defining the build number there:

	#define __BUILD 4

The script should run before compiling the source code, so let’s add the command in the prebuild section at Atmel Studio: 

	"$(MSBuildProjectDirectory)\deploy\prebuild.cmd" "$(MSBuildProjectDirectory)"

Now, all we need to do is to add “build.h” to our project and use `__BUILD` constant in any place of our code where we want to show the build number. It could be a welcome string on your debug console, a line in a log file on SD card, part of the advertising BLE message, tinny text on the screen during power on, etc. 

## Adapt it to your workflow
These scripts are just simple examples. Each developer has his own preferences; each project is different, and each team has a unique workflow. If this script doesn't work well with your workflow/project/team, you can always adapt it to what you need.
For example, maybe you don’t like the idea of creating tons of hex files on your disc. If so, you can edit the script only to make a copy on release builds:

	[SCRIPT GOES HERE]

# Final notes
Implementing a build script like this into your development process is just a beginning. There are other things and good practice you can implement to make release processes nice and smooth. 

## Release notes
Alright, let's be honest. Developers enjoy writing code, not documentation. And nobody likes to write release notes, but this is a must. It doesn't need to be something formal or extended. Just keep a file or Google Docs for the project and write a line or two each time you release a new version. At the end of the day, your team will be grateful for this. 

## Update your repository
I suppose that every developer in 2018 keeps his code in a repository (I’m scared to think there is somebody who doesn’t!). It’s a good practice to commit your code at the exact release point. Including a build number into commit message is a good idea too. 

## Track bugs and issues
Usually, when one finds a bug, it’s a good idea to fix it right away. But it’s not always possible, especially when the bug is reported by someone else. Keep track of all the bugs in one place so that you can come back to them later. It’s a good idea to be as detailed as possible, but even a vague description like “Sometimes Bluetooth drops after 30 seconds on build 139” is better than nothing. You don’t need to keep this in your head, and it gives you a chance to come back to it later and collect more information about the issue. 

# My team will never accept it
After the LED speed down episode, I talked to my manager. I explained what happened and suggested to implement some kind of release procedures to avoid this from re-occurring. “Nah, it’s a waste of time” was his response. “This customer is silly, if they don’t know how to use file explorer on their computer, it’s not our problem.”

Your team may not accept work with a well-defined release procedure. But nothing prevents you — embedded software developer to do so on your code. Nobody can prevent you to keep a `__BUILD` constant in your code that automatically increases on each build. Nobody can prevent you to include this number into a hex file name. You always can keep a release notes file for your own reference. And there is nothing wrong with keeping track of your bugs — at lease on a Spreadsheet.

After some time, your team will see the benefit and will accept to work this way. And if they don’t, maybe it’s time to find another job. After all, it's not just about being a developer, it's about being a great developer, so make sure you find a place where you can grow and get better every day. This is the reason why I started Firmware update - a blog about Embedded Systems Developing and C Programming. It's a place where I'm going to share my experience, ideas, and throughts I hope will help other developers to get better. 

Being a good developer is not a status, it's a processos. And one should never stop learning along the way. Becouse of this your opinion is important. How do yuo release your embedded software? What do yuo think about these scripts? What would you like to change / improve? Feel free to share your thought in the comment section.
