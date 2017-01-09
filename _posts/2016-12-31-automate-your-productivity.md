---
layout: single

categories: code

title:  "Automate Your Productivity"

excerpt: "Here's a dabble in automating the simple things by creating a daemon to handle all that for you."
---
# Automate Your Productivity
{% include toc %}
When I first created this blog, I had the unspoken goal of creating a blog post a month.
Well, it's been quite awhile since my last post - so it's safe to say I haven't kept that goal. (Who knew getting a puppy would offset your schedule so much??)
So while I've been hiatus, I've been researching the best ways to get productive. For awhile, I was using the Kaplan method with Trello, but I found I just wasn't getting things done. Everything felt crowded. I didn't have the comforting high-level view of the status _all_ my projects in one view.

Since I finally transitioned to Mac, I began using the most clamored task manager, OmniFocus. While I'm still adjusting my workflow, I've found that OmniFocus has both the complexity and simplicity of task managing that I need.

But there was one hiccup - some of my tasks comes from issues provided on other platforms. Now, I don't want to be manually entering tasks I already know I'm going to be working on - so why can't we automate that? We can easily do this through Zapier, but you don't nearly have the same control over the process as if you are doing it locally. So let's explore how to make that possible. While the below process is how I implemented the integration between OmniFocus and Pivotal Tracker, this can be abstracted to fit whatever automating needs you have.

*Caveat: This guide is geared towards macOS users. While these steps may be possible on Linux and Windows, you may need to substitute the launchd steps to something your OS supports.*

## Create Ruby Script
First, you need to create your Ruby script and save it in a safe place.
  For my script, I used the [omnifocus-pivotaltracker gem](https://github.com/vesan/omnifocus-pivotaltracker) gem. And the rest of the code is simple:

```ruby
require 'rubygems'
require 'omnifocus'

OmniFocus.sync NERD_FOLDER
```

  (NB: If you are using a gem that has an Applescript dependency, you need to use a Ruby version below 2.2.)

## Create an RVM alias for the script
Once your Ruby script is created, you will need to create an RVM alias for the launchd file, so that it can process the correct Ruby version for your script (otherwise, it'll default to the version on your local machine).

First, determine the Ruby version you need for your script. I'm using ruby 2.1.3.

So now, to create your RVM alias just run:

`rvm alias create my_script ruby-2.1.3@my_script`

Replace `my_script` and `2.1.3` with what's applicable for you.

## Create Bash script
Creating the bash script will allow the launch deamon to run the schedule program. If you want to forego the the launchd process all in general, and just want the program when you need to, this also suits that need.

Create an empty bash file like `my_daemon.sh` in `/usr/local/bin`. You'll now run your ruby script from the RVM alias created earlies.

You can retrieve your RVM path via `echo $rvm_path`.

So just include this line in your bash file: `/Users/user/.rvm/wrappers/my_script/ruby myfile.rb`, and save this file near your scripts. If your file requires arguments, append `-a somearg` at the end of the line.

Check and make sure your file ran: `bash /usr/local/bin/my_daemon.sh`

*NB: Ensure to have no blank lines as this may not make your file to run.*

## Create plist file
Lastly, all you need to do now is create your .plist file, which informs launchd where your bash script is and how often to run it. Below is a version of what I'm currently using.
You will save yours as `~/Library/LaunchAgents/com.yourdomain.projectname.plist`.

(Change `yourdomain` and `projectname`.)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>KeepAlive</key>
	<dict>
		<key>SuccessfulExit</key>
		<false/>
	</dict>
	<key>Label</key>
	<string>com.yourdomain.projectname</string>
	<key>ProgramArguments</key>
	<array>
		<string>/bin/bash</string>
		<string>/usr/local/bin/my_daemon.sh</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>StartInterval</key>
	<integer>3600</integer>
</dict>
</plist>
```
Besides editing where your file is located and label, you can adjust the `StartInterval`, which is defined in seconds.

Once you've saved your file, you'll need to register your .plist file via: `launchctl load ~/Library/LaunchAgents/com.yourdomain.projectname.plist`

You can manually start your script via `launchctl start ~/Library/LaunchAgents/com.yourdomain.projectname.plist`

There is a helpful GUI in managing all your daemons - [launch control](http://www.soma-zone.com/LaunchControl/), which I highly recommend as it's an easier interface to troubleshoot issues and customize your daemons.

Happy automatizing!
