# JP PIP, Conda, and Homebrew Notes
<BR>

***
From https://github.com/wblakecannon/conda-env
# Anaconda Environments
www.anaconda.com

## What is Anaconda?
Anaconda is a distribution of packages built for data science. It comes with conda, a package and environment manager. You'll be using conda to create environments for isolating your projects that use different versions of Python and/or different packages. You'll also use it to install, uninstall, and update packages in your environments. Using Anaconda has made my life working with data much more pleasant.

With Anaconda, it's simple to install the packages you'll often use in data science work. You'll also use it to create virtual environments that make working on multiple projects much less mind-twisting. Anaconda has simplified my workflow and solved a lot of issues I had dealing with packages and multiple Python versions.

Anaconda is actually a distribution of software that comes with conda, Python, and over 150 scientific packages and their dependencies. The application conda is a package and environment manager. Anaconda is a fairly large download (~500 MB) because it comes with the most common data science packages in Python. If you don't need all the packages or need to conserve bandwidth or storage space, there is also Miniconda, a smaller distribution that includes only conda and Python. You can still install any of the available packages with conda, it just doesn't come with them.

Conda is a program you'll be using exclusively from the command line, so if you aren't comfortable using it, check out this command prompt tutorial for Windows or our Linux Command Line Basics course for OSX/Linux.

You probably already have Python installed and wonder why you need this at all. Firstly, since Anaconda comes with a bunch of data science packages, you'll be all set to start working with data. Secondly, using conda to manage your packages and environments will reduce future issues dealing with the various libraries you'll be using.

## Managing Packages
Package managers are used to install libraries and other software on your computer. You’re probably already familiar with pip, it’s the default package manager for Python libraries. Conda is similar to pip except that the available packages are focused around data science while pip is for general use. However, conda is not Python specific like pip is, it can also install non-Python packages. It is a package manager for any software stack. That being said, not all Python libraries are available from the Anaconda distribution and conda. You can (and will) still use pip alongside conda to install packages.

Conda installs precompiled packages. For example, the Anaconda distribution comes with Numpy, Scipy and Scikit-learn compiled with the MKL library, speeding up various math operations. The packages are maintained by contributors to the distribution which means they usually lag behind new releases. But because someone needed to build the packages for many systems, they tend to be more stable (and more convenient for you).

## Environments
Along with managing packages, Conda is also a virtual environment manager. It's similar to virtualenv and pyenv, other popular environment managers.

Environments allow you to separate and isolate the packages you are using for different projects. Often you’ll be working with code that depends on different versions of some library. For example, you could have code that uses new features in Numpy, or code that uses old features that have been removed. It’s practically impossible to have two versions of Numpy installed at once. Instead, you should make an environment for each version of Numpy then work in the appropriate environment for the project.

This issue also happens a lot when dealing with Python 2 and Python 3. You might be working with old code that doesn’t run in Python 3 and new code that doesn’t run in Python 2. Having both installed can lead to a lot of confusion and bugs. It’s much better to have separate environments.

You can also export the list of packages in an environment to a file, then include that file with your code. This allows other people to easily load all the dependencies for your code. Pip has similar functionality with pip freeze > requirements.txt.

## Installing Anaconda
Anaconda is available for Windows, Mac OS X, and Linux. You can find the installers and installation instructions at https://www.continuum.io/downloads.

If you already have Python installed on your computer, this won't break anything. Instead, the default Python used by your scripts and programs will be the one that comes with Anaconda.

Choose the Python 3.6 version, you can install Python 2 versions later. (For Machine Learning Engineer Nanodegree you need Python 2 version) Also, choose the 64-bit installer if you have a 64- bit operating system, otherwise go with the 32-bit installer. Go ahead and choose the appropriate version, then install it. Continue on afterwards!

After installation, you’re automatically in the default conda environment with all packages installed which you can see below. You can check out your own install by entering conda list into your terminal.

## Managing Packages
Once you have Anaconda installed, managing packages is fairly straightforward. To install a package, type conda install package_name in your terminal. For example, to install numpy, type conda install numpy.

You can install multiple packages at the same time. Something like conda install numpy scipy pandas will install all those packages simultaneously. It's also possible to specify which version of a package you want by adding the version number such as conda install numpy=1.10.

Conda also automatically installs dependencies for you. For example scipy depends on numpy, it uses and requires numpy. If you install just scipy (conda install scipy), Conda will also install numpy if it isn't already installed.

Most of the commands are pretty intuitive. To uninstall, use conda remove package_name. To update a package conda update package_name. If you want to update all packages in an environment, which is often useful, use conda update --all. And finally, to list installed packages, it's conda list which you've seen before.

If you don't know the exact name of the package you're looking for, you can try searching with conda search search_term. For example, I know I want to install Beautiful Soup, but I'm not sure of the exact package name. So, I try conda search beautifulsoup.

It returns a list of the Beautiful Soup packages available with the appropriate package name, beautifulsoup4.

## Managing Environments
As I mentioned before, `conda` can be used to create environments to isolate your projects. To create an environment, use `conda create -n env_name` <list of packages> in your terminal. Here `-n env_name` sets the name of your environment (`-n` for name) and <list of packages> is the list of packages you want installed in the environment. For example, to create an environment named my_env and install numpy in it, type `conda create -n my_env numpy`.

When creating an environment, you can specify which version of Python to install in the environment. This is useful when you're working with code in both Python 2.x and Python 3.x. To my personal computer. I use them as general environments not tied to any specific project, but rather for general work with each Python version easily accessible. These commands will install the most recent version of Python 3 and 2, respectively. To install a specific version, use `conda create -n new_env python=3.3` for Python 3.3.

## Entering an Environment
Once you have an environment created, use `source activate my_env` to enter it on OSX/Linux. On Windows, use activate my_env.

When you're in the environment, you'll see the environment name in the terminal prompt. Something like (my_env) ~ $. The environment has only a few packages installed by default, plus the ones you installed when creating it. You can check this out with `conda list`. Installing packages in the environment is the same as before: `conda install <package_name>`. Only this time, the specific packages you install will only be available when you're in the environment. To leave the environment, type `source deactivate` (on OSX/Linux). On Windows, use `deactivate`.

## Saving and Loading Environments
A really useful feature is sharing environments so others can install all the packages used in your code, with the correct versions. You can save the packages to a YAML file with conda env export > environment.yaml. The first part conda env export writes out all the packages in the environment, including the Python version.

Above you can see the name of the environment and all the dependencies (along with versions) are listed. The second part of the export command, > environment.yaml writes the exported text to a YAML file environment.yaml. This file can now be shared and others will be able to create the same environment you used for the project.

To create an environment from an environment file use conda env create -f environment.yaml. This will create a new environment with the same name listed in environment.yaml.

## Listing Environments
If you forget what your environments are named (happens to me sometimes), use conda env list to list out all the environments you've created. You should see a list of environments, there will be an asterisk next to the environment you're currently in. The default environment, the environment used when you aren't in one, is called root.

## Adding Packages into Environment
You can specify the environment to install to in the install command

`conda install -n env-name package-name`

Or you can activate the environment, then install

`[source] activate env-name`
`conda install package-name`

## Removing Environments
If there are environments you don't use anymore, conda env remove -n env_name will remove the specified environment (here, named env_name).

## Exporting the Environment File
*NOTE: If you already have an environment.yml file in your current directory, it will be overwritten during this task.*

Activate the environment to export:

Windows:
`activate myenv`

macOS and Linux:
`source activate myenv`

NOTE: Replace myenv with the name of the environment.
Export your active environment to a new file:

`conda env export > environment.yml`

*NOTE: This file handles both the environment’s pip packages and conda packages.*

Email or copy the exported environment.yml file to the other person.

***





`which python` : command that shows you root directory of your default python install  

### Important Clarification about Virtual Environments regarding PIP and Anaconda:
##### Anaconda Virtual Env
When using a virtual env ("venv") in Anaconda (`conda create -n [venv name] python=X`), if you wish to install or modify packages in this venv, you must activate it first (`source activate [venv name]`) and then using `conda` *only* can you modify the packages in this environment.  

**Note:**  
Using the command above to create the venv with `anaconda` appended to the end (ex: `conda create -n [venv name] python=X anaconda`) will create a new venv with the _full Anaconda distribution already installed_.  From here, you can modify the version of a given package.  This approach creates a venv with way more packages than are usually going to be needed, of course, making it fairly wasteful space-wise.  In general we will want to create a blank venv and then populate it with the specific packages of interest for our project.  However, I wanted to note this option.  

##### PIP Virtual Env ("Vanilla")
The below quote is from the [Conda cheatsheet](https://conda.io/docs/_downloads/conda-cheatsheet.pdf), being in an active venv and installing a package via `pip` does indeed install that package only into the active venv and not into your global Python installation.  (This is a good thing).

> Install a package directly from PyPI into the current active environment using pip.  
Ex: "pip install new_module" installs new_module into current active environment only.




For either venv, when done you must deactivate the env, which returns you to your default global Python installation.
+ `deactivate [venv name]` (vanilla)
+ `source deactivate [venv name]` (conda)

To delete a venv, you can remove its folder.  For a conda venv, you can use a conda command to cleanly remove it as well.
+ `rm -rf [venv name]` (or visually in the Finder)
+ `conda remove -n [venv name] --all`


See more about using [PIP venv](http://docs.python-guide.org/en/latest/dev/virtualenvs/) and [Anaconda venv](https://conda.io/docs/using/envs.html)



### Conda commands
Command | Action
--------|--------
`conda list`

### pip commands
Command | Action
--------|--------
`python --version` | gives current version of python
`pip install [package]==X.x.x` | force install a specific version of a package, downgrading if necessary.
`pip install [package] --upgrade` | upgrades a package, overwriting as needed


Common packages used (beware of cross-dependencies!):  
+ numpy
+ pandas
+ matplotlib
+ flask
+ scipy
+ scikit-learn
+ scikit-image
+ seaborn
+ selenium
+ six
+ SQLAlchemy
+ Werkzeug
+ numexpr
+ pymc
+ psql
+ psutil
+ psycopg2
+ pymongo
+ bokeh
+ beautifulsoup4
+ Flask
+ ghp-import (*github pages* for *Pelican* static site publishing)
+ sphinx
+ ipython
+ Jinja2
+ jupyter
+ mrjob
+ nbformat

### Current list of virtual environments I have set up:
`conda info --envs`
Environment | Python Version | Purpose
------------|---------|---------


### Switch between two Python environments (2.7 and 3.x) in Anaconda:
My default installation is (currently) Python 2.7.
I installed Python 3 and named it `python3`, which is the name we will use to utilize that environment.  Type the following commands into the terminal to start or stop a Python 3 environment.  During DSI, we installed a GraphLab environment called `gl-env`.
(Some of these commands may take `--` instead of `-` -- I can't tell yet)
(All commands are one-line commands, regardless of how they are line-wrapped below).
Command | Effect
--------|--------
`conda env -help` | List of all Conda commands
`conda create -name [your_env_name] python=X.x anaconda` | Creates the virtual env. <br> For Python version, can use general version [`2` or `3`] or specify point version [e.g. `2.7` or `3.5`, etc.]
`source activate [env name]` | Activates virtual env
`source deactivate [env name]` | Deactivates virtual env
`conda info -envs` | Verify current environment
`conda env list` | Lists all virtual env
`conda create -name [new_name] -clone [env_to_clone]` | clone an existing env with a new name.
`conda remove -name [env_name] -all` | removes an env. Verify its removal with `conda info -envs`

TIP: Many frequently used options after two dashes (--) can be abbreviated with just a dash and the first letter. So --name and -n options are the same and --envs and -e are the same. See conda --help or conda -h for a list of abbreviations.

**Share an environment**
You may want to share your environment with another person, for example, so they can re-create a test that you have done. To allow them to quickly reproduce your environment, with all of its packages and versions, you can give them a copy of your environment.yml file.

[This guide](https://uoa-eresearch.github.io/eresearch-cookbook/recipe/2014/11/20/conda/) is the best nano guide on creating a virtual env in Conda.

See [this page](https://conda.io/docs/using/envs.html) for more info on advanced environment handling, such as exporting, creating env by hand, variable scripts, and building identical conda envs.

Also see Anaconda's good [write up](http://conda.pydata.org/docs/py2or3.html) for a short guide on using environments to switch between different versions of Python if needed:  


##### Installing Packages with Anaconda
Anaconda Python and Packages

We use the Anaconda scientific python stack which is just a vanilla version of Python 2.7 along with all the packages that a data scientist would need, including NumPy, SciPy, SciKit-Learn, Pandas, and matplotlib. Anaconda manages the Python environment for us. If you need to install other Python packages (unlikely), do so with the conda command-line utility (i.e. `conda install some-cool-package`). Use conda list to see what's installed.

to update itself `conda update conda`


Note we can create a list of packages we want to use in an environment and then run a command which installs all these into the desired env.  There are a couple ways to do this.  One is by using `pip freeze` to create a `.txt` output file of all packages and versions currently installed in a given environment.  I'm not currently sure how to use that `.txt` file to install everything in it into an env, but I am certain it can be done.

Another way is to use a `.yml` file.  We can include all desired info in the `.yml` file, even the env name.  Example:

```yaml
name: MyYAMLEnv
channels:
- defaults
dependencies:
- python=3.6
- numpy
- pandas
- ipython
- ipython_genutils
- matplotlib
- jsonschema
- openssl
- pip
- pytest
- scikit-learn
- setuptools
- statsmodels
- scipy
- mkl
pip:
    - ipython_genutils

# etc...
```

Then we can issue a bash command to create this specified env in conda.  For example:  

`conda env create -f MyYAMLEnv.yml`  

The `-f` argument says to _force_ install all dependencies, even if already installed.


<BR>
<BR>
<BR>





## Homebrew Package Manager
***
<BR>

Homebrew is a Mac OS package manager, and Homebrew **Cask** is a full-application manager (Google Chrome, for example).  We will commonly use **brew** to install packages which are not available through **conda** or **pip**.  **Homebrew** is built with Ruby and Git, so it supports versioning in a sense and supports installing packages that are *not* Python-specific, unlike **pip** or **conda** (though technically I believe **conda** does support some non-Python packages, it is certainly Python-centric).  Homebrew installs packages to their own directory and then symlinks their files into `/usr/local`.  Full documentation links are below.  

### Homebrew Installation
Install **homebrew** and brew **cask**:
  + Install homebrew. Instructions [here](http://brew.sh/)
    + ...Or paste the following into your terminal to install:  
    `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
  + Install brew cask with: `brew install caskroom/cask/brew-cask`
  + If you already have them, update: `brew update && brew upgrade brew-cask`
<BR>


### Updating BASH (or ZSH) with Homebrew
Here are three links with varying methods of command line attempts at updating Bash.  I think the middle one is the clearest but I couldn't get the `sudo` commands to use the correct version of Bash, though the `/usr/local/bin/bash` dir IS correct.  Hmmmm.  Anyway, try all three.
https://coderwall.com/p/dmuxma/upgrade-bash-on-your-mac-os
https://johndjameson.com/blog/updating-your-shell-with-homebrew/
https://github.com/Toberumono/Miscellaneous/wiki/Installing-Bash-4.3-on-Mac-OSX

Here's the gist:
Apple uses an older version of Bash for [some licensing reasons](https://apple.stackexchange.com/questions/193411/update-bash-to-version-4-0-on-osx) (it seems their hands are tied, legally).
They use version 3.2.57, which came out in the oughts.  When we open __Terminal__, it loads this old Apple-installed Bash file (by default at `/bin/bash`).
Current versions of Bash are in the 4.4.x range and offer many enhancements and, perhaps most important, import security improvements and bug fixes.  
We can install the newest version of Bash using Homebrew.

`brew install bash`

This does _not_ overwrite the base installation of Bash that comes with your Mac.  Instead, like all Brew packages, it installs the package in its own directory and makes a symlink to `/usr/local/bin/`.  Cool, now we just need to make the Terminal use _that_ version of Bash instead of the older Apple default.  There are various ways to do this, but the primary two that make sense to me are:
1. Use command line:
    `sudo -s`
    `echo /usr/local/bin/bash >> /etc/shells`
    `chsh -s /usr/local/bin/bash`

2. Use System Preferences:
    system preferences ->
    Users & Groups (unlock pref pane) ->
    (right click on your account) Advanced Options ->
    change Login shell option to `/usr/local/bin/bash`

I couldn't get \#1 to work, but it should work because it uses the correct directories.  For some reason everytime I went into `sudo` it reverted to the old version of Bash.
But I can _confirm_ that using method \#2 DID work for updating Bash.


#### Git tab complete and branch info in terminal:
https://stackoverflow.com/questions/12870928/mac-bash-git-ps1-command-not-found






### Homebrew Commands
##### Common Commands
Command	| Description
--------|------------
`brew install [package]`	| Install a package
`brew upgrade [package]`	| Upgrade a package; leave blank to upgrade all (caution)
`brew unlink [package]`	| Unlink
`brew link [package]`	| Link
`brew switch [package] 2.5.0`	| Change versions
`brew list --versions [package]`	| See what versions you have
`brew info [package]`	| List versions, caveats, etc
`brew cleanup [package]`	| Remove old versions, include `-n` to see which packages would be removed
`brew edit [package]`	| Edit this formula
`brew home [package]`	| Open homepage
`brew pin [package]`  | Stop from upgrading/updating
`brew unpin [package]`  | Allow upgrading/updating
<BR>

##### Global Commands
Command	| Description
--------|------------
`brew update`	| Update brew
`brew list`	| List installed
`brew outdated`	| What’s due for upgrades?
<BR>

##### The Rest
Command	| Description
--------|------------
`man brew` | Display Homebrew User Manual
`brew --cache`	| Print path to Homebrew’s download cache (usually `~/Library/Caches/Homebrew`)
`brew --cellar`	| Print path to Homebrew’s Cellar (usually `/usr/local/Cellar`)
`brew --config`	| Print system configuration info
`brew --env`	| Print Homebrew’s environment
`brew --prefix`	| Print path to Homebrew’s prefix (usually `/usr/local`)
`brew --prefix [formula]`	| Print where formula is installed
`brew audit`	| Audit all formulae for common code and style issues
`brew cleanup [formula]`	| Remove older versions from the Cellar for all (or specific) formulae1
`brew create [url]`	| Generate formula for downloadable file at url and open it in `$HOMEBREW_EDITOR` or `$EDITOR2`
`brew create [tarball-url] --cache`	| Generate formula (including MD5), then download the tarball
`brew create --fink [formula]`	| Open Fink’s search page in your browser, so you can see how they do formula
`brew create --macports [formula]`	| Open MacPorts’ search page in your browser, so you can see how they do formula
`brew deps [formula]`	| List dependencies for formula
`brew doctor`	| Check your Homebrew installation for common issues
`brew edit`	| Open all of Homebrew for editing in TextMate
`brew edit [formula]`	| Open [formula] in `$HOMEBREW_EDITOR` or `$EDITOR`
`brew fetch --force -v --HEAD [formula]`	| Download source package for formula; for tarballs, also prints MD5, SHA1, and SHA256 checksums
`brew home`	| Open Homebrew’s homepage in your browser
`brew home [formula]`	| Opens formula’s homepage in your browser
`brew info`	| Print summary of installed packages
`brew info [formula]`	| Print info for formula (regardless of whether formula is installed)
`brew info --github [formula]`	| Open Github’s History page for formula in your browser
`brew install [formula]`	| Install formula
`brew install --HEAD [formula]`	| Install the HEAD version of formula (if its formula defines HEAD)
`brew install --force --HEAD [formula]`	| Install a newer HEAD version of formula (if its formula defines HEAD)
`brew link [formula]`	| Symlink all installed files for formula into the Homebrew prefix3
`brew list [formula]`	| List all installed files for formula (or all installed formulae with no arguments )
`brew options [formula]`	| Display install options specific to formula
`brew outdated`	| List formulae that have an updated version available (`brew install formula` will install the newer version)
`brew prune`	| Remove dead symlinks from Homebrew’s prefix4
`brew remove [formula]`	| Uninstall formula
`brew search`	| List **all** available formula
`brew search [formula]`	| Search for formula in all available formulae
`brew search /[formula]/`	| Search for `/formula/` (as regex) in all available formulae
`brew test [formula]`	| If formula defines a test, run it
`brew unlink [formula]`	| Unsymlink formula from Homebrew’s prefix
`brew update`	| Update formulae and Homebrew itself
`brew upgrade`	|Install newer versions of outdated packages
`brew upgrade [formula]`	| Install newer version of formula
`brew versions [formula]`	| List previous versions of formulae, along with a command to checkout each version  

You can update outdated packages with any of the following:
+ `brew upgrade`
+ `brew install 'brew outdated'`
+ `brew outdated | xargs brew install`  

You can uninstall Homebrew entirely with the following:  
`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"`  
Download the [uninstall script](https://raw.githubusercontent.com/Homebrew/install/master/uninstall) and run `./uninstall --help` to view more uninstall options.  

If you do not uninstall all of the versions that Homebrew has installed, Homebrew will continue to attempt to install the newest version it knows about when you do (`brew upgrade --all`). This can be surprising.

To remove a formula entirely, you may do:  
`brew uninstall formula_name --force`  
Be careful as this is a destructive operation.




*1.* To delete a specific version, just go to the folder in the Cellar and `rm -rf `it; alternatively, drag it to the trash in Finder.

*2.* Homebrew tries to guess the formula name and version. If it fails, you’ll have to make your own template. I suggest copying `wget`’s.

*3.* Symlinking is automatically performed when installing formulae. It’s useful for DIY installation, or swapping out versions of a package that has multiple installs.

*4.* This is generally not needed. However, it can be useful if you are doing DIY installations.
<BR><BR>



### Cask Commands
Command | Action
--------|-------
`brew cask search` | List all available casks
`brew cask list` | List installed casks
`brew cask search <cask-name>` | Search a given cask (e.g. `chrome`)
`brew cask install <cask-name>` | Install a given cask (e.g. `google-chrome`)
`brew cask uninstall <cask-name>` | Uninstall a given cask
`brew cask info <cask-name>` | Display information of a given cask
`brew cask home <cask-name>` | Open homepage of a given cask (this will open the application's homepage)
`brew update` | Update (get the latest casks but does not upgrade Homebrew Cask itself, nor your applications)
`brew upgrade brew-cask` | Upgrade Homebrew Cask itself
`brew cask checklinks` | Check for bad cask links:
`brew cask alfred` | Modify Alfred's scope to include the Homebrew Cask apps:
`brew cask create <cask-name>` | Create a cask of the given name and open it in an editor:
`brew cask edit <cask-name>` | Edit a given cask
`brew cask audit` | Verify install-ability of all available casks
`brew cask audit <cask-name>` | Verify install-ability of a given cask:
<BR>


### Homebrew & Cask Docs
+ [Homebrew home page](brew.sh)
+ [Cask home page](http://gillesfabio.github.io/homebrew-cask-homepage/)
+ All [Official Homebrew docs](http://docs.brew.sh/)
+ [FAQ](http://docs.brew.sh/FAQ.html)
+ [Tips and Tricks](http://docs.brew.sh/Tips-N'-Tricks.html)
+ [Homebrew Github Repo](https://github.com/Homebrew/brew)
+ See [Common Issues](http://docs.brew.sh/Common-Issues.html) if you are having problems, including after upgrading Mac OSX:
  + Upgrading macOS can cause errors like the following:
    + `dyld: Library not loaded: /usr/local/opt/icu4c/lib/libicui18n.54.dylib`
    + `configure: error: Cannot find libz`  

    Following an macOS upgrade it may be necessary to reinstall the Xcode Command Line Tools and `brew upgrade` all installed formula:  
    ```bash
    xcode-select --install
    brew upgrade
    ```

+ Full [installation instructions](https://github.com/Homebrew/brew/blob/master/docs/Installation.md#installation), including a note about installing in a unrecommended directory.  If you follow the installation instructions above, you should be fine and have no need for this.
> However do yourself a favor and install to /usr/local. Some things may not build when installed elsewhere. One of the reasons Homebrew just works relative to the competition is because we recommend installing to /usr/local. Pick another prefix at your peril!

+





<BR>
<BR>
<BR>

## PIP Package Manager
***
<BR>

### PyPI - the Python Package Index
Using **PIP**, packages are usually installed from the Python Package Index (PyPI -- said "Pie-Pee-Eye" or "Pie-Pie").  PyPI is a repository of software for the Python programming language.  To install a package from the Python Package Index, just follow the instructions below.  
<BR>

### PIP Installation
Installing PIP is easy and if you're running Linux, its usually already installed.

If it's not installed or if the current version is outdated, you can use the package manager to install or update it.  
On Debian and Ubuntu: `$ sudo apt-get install python-pip`  
On Fedora:  `$ sudo yum install python-pip`  
If you are using Mac, you can simply install it through **easy_install**:  `sudo easy_install pip`
<BR>

### PIP Commands
Just typing `pip` in your terminal brings up the help menu. The most common usage for **pip** is to install, upgrade or uninstall a package.

Command | Action
--------|-------
`install [package]`	| Install packages.
`uninstall [package]` | Uninstall packages.
`freeze [package]` (optional) | Output installed packages in requirements format.
`list` | List installed packages.
`show [package]` | Show information about installed packages (can list multiple packages, separated by a space).
`search [package]` | Search PyPI for packages.
`zip [packages]` | Zip individual packages.
`unzip [packages]` | Unzip individual packages.
`bundle` | Create pybundles.
`help` | Show help for commands.

Below is a simple example of how to use **pip** to search for a package named **Flask**, install it, inspect it, and uninstall it.
<BR>

1. Search - `pip search Flask`
2. Install - `pip install Flask`
3. Inspect - `pip show Flask`  
4. Uninstall - `pip uninstall Flask` (if needed)

**JP Note:**  
**Flask** and many, many other packages are included in the Anaconda distribution of Python.  So there is no need to manage them from **pip** in most cases.  If you use virtual environments, it is recommended to use **conda** virtual environments and to manage packages within those venvs via **conda** as much as possible.  However, you can use **pip** to install a given package into a specific Anaconda venv just as you normally would any venv.
<BR>
