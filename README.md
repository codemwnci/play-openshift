play-openshift
==============

Action Hooks for deploying Play Framework 1 to OpenShift as a DIY cartridge

The following is a step by step guide on how to deploy a Play Framework 1 application, natively, to OpenShift. 

In a nutshell, the steps are
<ol>
        <li>Sign up to OpenShift</li>
	    <li>Install Ruby and GIT (I deployed from Windows, so used Ruby for Windows and GIT for windows)</li>
        <li>Install RHC (red hat console) using the Ruby Gem -- <code>gem install rhc</code></li>
        <li>Create an application on OpenShift using a DIY cartridge</li>
        <li>Put your application code in the cloned directory</li>
        <li>Copy the open-shift hooks (see code below) into the hooks directory, which will install Play on the DIY cartridge, and correctly start/stop Play when needed</li>
        <li>Set up the application.conf to work with OpenShift ports/db/etc</li>
        <li>GIT Add/Commit/Push, to push the code to OpenShift, and watch the magic happen!</li>
</ol>

Step 1 - 3 will only ever need doing once. Once configured, each modification simply needs a GIT add/commit/push to see your changes running (Step 8). Step 4 to 7 should be a case of copy/paste. So, hopefully, we can have you up and running in no time.

<h4>The Detail</h4>

<b>Steps 1, 2 and 3 - SignUp and Install Software</b>
I won't go into detail on Step 1 to 3, as the OpenShift information does a very good job of doing so. So, follow <a href="https://openshift.redhat.com/community/get-started#cli" target="_blank">steps 1, 2 and 3 on the OpenShift Setup page</a> and install the necessary tools for your platform (Ruby, GIT and RHC) plus sign up for an account and set up RHC.


<b>Step 4 - Create an application</b>
Before running the following command, make sure a directory with the same name of the app you want to create doesn't already exists, otherwise the git clone will fail. Once you are happy, run the following code from the command prompt.

    rhc app create #appname# diy-0.1

Replacing #appname# with the name of your application. This will create a DIY cartridge for you. This will automatically call GIT clone, to clone the application on to your local machine. If for some reason you have created an application using the Web site at OpenShift, you can use <code>git clone</code> to clone the repo to you local machine, but in our example, you don't need to do this.

If you want a Database (which was kind of my whole reason for coming to OpenShift), then follow this command with

    rhc cartridge add mysql-5.1 -a #appname#

Again, replace #appname# with your application name you set up in the previous command.


<b>Step 5 - Develop your Play application</b>
Store all your play application code inside the cloned repository, in the standard Play structure. If you have already started a play app, and are just looking to deploy it, you can safely just copy all the play code into the cloned repo, so that app, public, conf, lib etc are all at the same level as the .openshift directory. If you are starting a new application, create a new application with a different name (otherwise Play will complain a directory already exists), and copy the contents of the new application into the cloned repo.

You directory structure should look something like

    #appname#
    |+ .openshift
    |+ app
    |+ conf
    |+ diy
    |+ lib
    |+ misc
    |+ public

You can still run <code>play run</code> from this structure as you did before.

<b>Step 6 - Get the open_shift hooks!</b>
The open shift hooks are a set of bash scripts that are executed at certain stages of the server's lifecycle. It has scripts for things such as pre-build, start, stop etc. These are the scripts that will, install Play if it doesn't already exist (so it will only install it once, unless you specify a new version), start the server and shutdown the server when OpenShift spins up and spins down instances. This is the bulk of the work for an OpenShift DIY cartridge. Fortunately, you can copy the code verbatim.

When you created the app, and the repo was cloned, a set of empty action_hooks were created and downloaded. There should have been 6 in total. These are
<ul>
  <li>Build       - We can leave this blank</li>
  <li>Deploy      - We can leave this blank</li>
  <li>Post-Deploy - We can leave this blank</li>
  <li>Pre-Deploy  - This is where we will download and configure Play</li>
  <li>Start       - This will start a Play instance</li>
  <li>Stop        - This will stop a Play instance</li>
</ul>

So, we have 3 scripts we need to populate, and to make things easier, we will also create a shared script to help set up the configuration variables which will allow us to get data from our Play application.conf as well. These scripts were written by opensas, but I have updated them as there have been a few platform changes since his version was published.

Copy the following code into the action_hooks directory, and we are almost good to go!

Go to <a href="https://github.com/codemwnci/play-openshift" target="_blank">https://github.com/codemwnci/play-openshift</a>, and copy the files into your action hooks directory.

NOTE: In the start script, I have configured the timezone to be for London (as OpenShift is US based, and most of my web apps are for UK based). If you don't care about the default timezone, just delete the 

    -Duser.timezone=Europe/London
	
in the start script, or if you want a different timezone, update the value before you deploy your application.


<b>Step 7 - Set up the application conf</b>
Inside the scripts above, Play is configured to start with the ID of openshift. This means that we can keep our openshift config and local config completely separated. So, inside of your application.conf, place the following pieces of config.

    # Openshift id configuration
    # ~~~~~
    %openshift.application.mode=prod
    %openshift.http.port=${OPENSHIFT_INTERNAL_PORT}
    %openshift.http.address=${OPENSHIFT_INTERNAL_IP}

    # openshift mysql database
    # ~~~~~
    %openshift.db=${OPENSHIFT_MYSQL_DB_URL}#appname#
    %openshift.jpa.ddl=update

Replace #appname# with the name of your application (the same name as you set up in step 4). This allows Play to use the short db configuration, rather than the 4 individual parameters.


<b>Step 8 - Commit and Deploy</b>

Finally, we are ready to deploy our app. So quite simply we run three simple GIT commands.

    cd #appname#
	git add .
    git commit -m "deploy to OpenShift"
    git push


If you want to see the log files for your app running, you can tail the logs by running

    rhc tail #appname#

Again, replace #appname# with the application name set up in step 4.

<h4>Enjoy</h4>
I hope this saves you hours of effort. The time it has taken me to type this up hopefully will be less than the time it would take you to figure all this out through trial and error, like I did. Enjoy.