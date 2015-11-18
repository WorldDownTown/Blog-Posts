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

##Common Carthage Workflow (I'll Show you the ropes kid, show you the ropes!)
#### Create a cart file with dependencies listed and optionally branch info
Cart file grammar:

```bash
# This is Comment
# The first keyword is either 'git' for a repository hosted by a non-github server 
# or 'github' for a repository hosted on github with the format 'github "Username/Repository name" (optional) "[branch name]" OR "== / >= / ~> [VERSION_NUMBER]"
# '==' and >= mean what you think they mean and '~>' means "Compatible With"
github "ReactiveCocoa/ReactiveCocoa" "master"
```
