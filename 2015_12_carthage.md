# Carthage: The non-invasive dependency manager

## A Bird's eye view
Carthage is a decentralized dependency manager for iOS and OS X frameworks (not static libraries).

Unlike CocoaPods, Carthage has no central location for hosting repository information (like pod specs), dictates nothing as to what kind of project structure you should have aside from optionally having a default ```Carthage/``` folder in your project's
root folder that internally houses built frameworks in ```Build/``` and optionally source files in ```Checkouts/``` if building directly from source.
This folder hierarchy is automatically generated after running the command ```bash carthage bootstrap```)
Carthage leaves it open to the end user as to how they want to manage third party libraries, either by having both the ```Cartfile``` and ```Carthage/*``` folder checked in under source control 
(in which case the user does not have to run any carthage commands) or just the Cartfile listing the frameworks you wish to build and include as part of your project.
under source control.
Since there is no centrailized source of information project discovery is more difficult with carthage but other than that normal operation is simpler and less error prone. 

##Setup

####Install via Homebrew
Carthage is most easily installed via the community built OS X packaged manager [HomeBrew](http://brew.sh/)
Luckily a build formula already exists, therefore if you have Homebrew installed you only need to run the command:
```shell brew install carthage```

####Install via Release Download
In the event that you do not want to use Homebrew you are still in luck. 
Just simply download the appropriate ```carthage.pkg``` from the [Releases Page](https://github.com/Carthage/Carthage/releases)

####Setup Instructions for the Hardcore / Early Adapters 
If you really want to be on the bleeding edge (and live on the edge a little) you can install the most up to date version 
by cloning the master branch and running the make command:
```shell
git clone git@github.com:Carthage/Carthage.git
cd Carthage
make install
```

##Common Carthage Workflow 
#### Create a cart file with dependencies listed and optionally branch or version info
Cart file grammar:

```bash

# The first keyword is either 'git' for a repository hosted by anything not github  
# and 'github' for a repository hosted on github with the format 'github "Username/Repository name" (optional) "[branch name]" OR "== / >= / ~> [VERSION_NUMBER]"
# '==' and >= mean what you think they mean and '~>' means "Compatible With"
github "ReactiveCocoa/ReactiveCocoa" "master" #Latest version of the master branch of reactive cocoa
github "rs/SDWebImage" ~> 3.7 # Version 3.7 and versions compatible with 3.7
github "realm/realm-cocoa" == 0.96.2 #Only use version 0.96.2

```

##Basic Commands
Assuming that all went well with the install step, you now should be able to simply run ```bash carthage bootstrap```
and watch carthage go through the Cartfile one by one and fetch the frameworks (or build them after fetching from source if using --no-use-binaries)
Given that this goes without a hitch all that is left to do is to add a new run script phase to your target.
To do this just simply click on your target in XCode and under the 'Build Phases' tab click the '+' button and select "New Run Script Phase"

Type this in the script section:

```bash 
/usr/local/bin/carthage copy-frameworks 
```

And then below the box where you typed the commands to run add an input file per framework that you wish to use.

#Last but not least 
Once again click on your target and navigate to the General tab, then go to the section ```Linked Frameworks and Libraries``` and add the .frameworks from ```[Project Root]/Carthage/Build/iOS/*``` to your project. 

From this point onward everything should build and run just fine.
As project requirements change and you wish to add / remove frameworks or upgrade versions just edit the ```Cartfile``` run the command ```carthage update``` and if needed add / remove the frameworks from your project settings.

Its that simple.

#Notes on Source Control with Carthage

Given that all of your project's thirdparty source and frameworks are located under the ```Carthage/``` folder, in my experience it is much easier to just simply place this folder under source control. 
The merits of doing so are simple, when cloning the project or switching branches there is no need to run ```carthage bootstrap``` or ```carthage update```. 
This saves a considerable amount of time and the only expense for doing so is the overall size of the repository will increase. 


