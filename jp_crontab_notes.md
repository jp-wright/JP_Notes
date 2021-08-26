# Cron Job Notes

##### Table of Contents
<!-- TOC -->
- [Table of Contents](#table-of-contents)
- [What Is Cron?](#what-is-cron)
- [What Is Cron Used For?](#what-is-cron-used-for)
- [Cron Syntax](#cron-syntax)
    - [Confirming the list of Cron Jobs](#confirming-the-list-of-cron-jobs)
- [Helpful Tips](#helpful-tips)
    - [Cron Email](#cron-email)
    - [Debugging - Failure to Execute](#debugging---failure-to-execute)
    - [Shebang](#shebang)
    - [VIM (Default)](#vim-default)
    - [Nano](#nano)
    - [Full Path of Command Executables](#full-path-of-command-executables)
    - [Escaping Special Characters in File Paths](#escaping-special-characters-in-file-paths)
    - [Executing Python](#executing-python)
- [Common Crontab Problems](#common-crontab-problems)
    - [PATH Problems](#path-problems)
        - [Inside the Shell Script](#inside-the-shell-script)
        - [Inside Your Wrapper Shell Script](#inside-your-wrapper-shell-script)
    - [Unescaped Characters in File Path](#unescaped-characters-in-file-path)
    - [User Access - Change Permissions](#user-access---change-permissions)
    - [User Access - Root](#user-access---root)
    - [MacOS App Permissions](#macos-app-permissions)
- [References](#references)
- [My Cron Jobs](#my-cron-jobs)

<!-- /TOC -->



## What Is Cron?
Here's my high level understanding of the cron job (short for "chronos," though strangely without the "h".  As any RPG fan of the 1990s knows, "chronos" is Greek for "time") functionality in Unix.  (I'm using a Mac, for what it's worth).

`cron` is a daemon that is always running.  That means it's just a process that keeps operating in the background.  The purpose of this process is to execute a list of commands at regular intervals.  That's it, nothing more.

The list of commands is called the `crontab` (for _cron table_) and is blank by default.  
To see the list (table) and edit it, we need to enter `crontab -e` in the terminal.  (The default editor is VIM, see below for commands).  This will open the table where we can add commands to be executed by the cron daemon.  Commands can either be typed directly into the `crontab` or can be pasted (yay).

The table of commands is executed in order of appearance, top to bottom.

<BR>

## What Is Cron Used For?
`cron` is used for one thing: automating tasks.  For my purposes, it is to automate downloading of data, executing Python or Shell scripts on that downloaded data, and updating environments / repositories.

<BR>

## Cron Syntax
Cron has five time intervals the user can define:
+ Minutes of an hour
+ Hours of a day
+ Day of month
+ Month
+ Day of week [0 to 7, 0 _and_ 7 both equal 'Sunday']

The syntax itself works like this: a value for each of the time variables is entered, separated by a space.  If the time unit is not specified, we use an asterisk (`*`) to say "please execute at all intervals of this time variable."  So, if it were the _minutes_ variable, the cron job would execute every minute of the specified hours.

The actual command to be executed (i.e. the script you want to run) is listed on the same line after the time specification values.  

```
* * * * * [your_script_to_execute.sh]  <-- this line runs every minute of every day.
8 * * * * [your_script_to_execute.sh]  <-- runs on the 8th minute of every hour.
8 17 * * * [your_script_to_execute.sh]  <-- runs at 5:08 pm every day. (17 = military time).
8 17 6 * * [your_script_to_execute.sh]  <-- runs at 5:08 pm on the 6th day each month.
```

...You get the idea.  There are more complex time intervals which can be used.  For help with these, consult the really good [Wiki page](https://en.wikipedia.org/wiki/Cron) or this [more thorough guide](https://phoenixnap.com/kb/set-up-cron-job-linux)

If you are struggling to get the time interval syntax desired, use this [excellent helper site](https://crontab-generator.org/) to generate it for you.

__Important:__ Every job in the `crontab` exists on a single line.  `crontab` does not recognize any line-continuation characters, so any single command that is split over multiple lines will be interpreted as a list of __multiple cron jobs__ (and thus, likely fail to execute).


##### Confirming the list of Cron Jobs
Enter `crontab -l` in the terminal to display the list of active cron jobs.


<BR>
<BR>

## Helpful Tips
##### Cron Email
The crontab can support an outgoing email address to email upon completion of a cron job.
I'm not sure if a different email can be used per each job or only one per entire crontab (I believe it is only one per crontab).

Include the following outgoing email syntax at the start of your crontab.
`MAILTO="john.doe@domain.com"`

Note: I haven't had this work yet.  It's supposed to use the email client attached to your system-user profile that you're executing the cron job from, but mine isn't working.  Clearly have to set up some other machinery to utilize email.


##### Debugging - Failure to Execute
__IMPORTANT:__ If a cron job fails to execute, you will __not__ be alerted in any manner by default.  It is a completely silent failure.  This means you have to do something else to be alerted to what's going on in your cron job.

1. If your cron job is meant to create some output, such as downloading a new file or creating some new version of an existing file, you can obviously check to see if these files were created or updated by their `date modified` information.

2. Capture the cron job output in a log file.  This is easy and is the best way to see what's going on with your cron jobs.  The `2>&1` closing command tells the cron job to output both the stdout and stderr.
    ```
    * * * * * [your_script_to_execute.sh] > /path/to/cron_output.log 2>&1
    ```
    It's standard practice to store crontab log files in `/tmp/`.  Personally, I name the log files after each specific job, so I know what to investigate if something is amiss, such as:
    ```
    * * * * * [your_script_to_execute.sh] > /tmp/cron_<specific_file_name>_output.log 2>&1
    ```

##### Shebang
Including a shebang at the top of your scripts which are called in the crontab makes the script executable.  I have not personally tested the success of cron jobs with vs. without the shebang.  I just include it by default.

+ Shell: `#!/bin/sh`
+ Bash: `#!/bin/bash`
+ post-2021 Anaconda distro of Python (good): `#!/opt/anaconda3/bin/python`
+ pre-2021 Anaconda distro of Python: `#!/Users/<username>/anaconda3/bin/python`
+ Default Python install (v2.x on old MacOS -- not good): `#!/usr/bin/python`


##### VIM (Default)
The default editor when using `crontab -e` in the terminal is VIM.  
+ `i` beings "insert" mode which allows the user to enter or delete text. Leave "insert" mode by hitting `escape`.
+ `:w` saves ("writes") the crontab file.
+ `:q` exits VIM.
+ `:wq` saves then exits VIM.
+ `dd` deletes the line the cursor is on.


##### Nano
I'm not a VIM user and find the control scheme to be frustrating.  As a result, I prefer a more traditional text editor like _nano_.  To make your crontab load in nano, use the following command in terminal.

```
export EDITOR=nano; crontab -e
```

##### Full Path of Command Executables
When executing a script, we might need to preface the script itself with the command to launch/use it.  Since cron is executed form a different environment than the environment in which our script (whatever it may be) lives in, it is smart to tell cron _exactly_ what we want to use to run our script (Python, bash, shell, etc...).

For example, we wouldn't just use the following to execute a bash script:

```
* * * * * bash my_shell_script.sh
```

The potential weakness here is the keyword `bash` -- this command as written here is actually just an _alias_ for the full path to the `bash` executable.  And if our cron job is not being called from an environment which knows what the simple `bash` alias is pointing to, it will fail to execute.

Instead, we want to specify the full path as follows:

```
* * * * * /bin/bash my_shell_script.sh
```

This applies to other commands, not just `bash`.  It is simply used as an example, here.


##### Escaping Special Characters in File Paths
Any executable path that has special characters, like parentheses `(` or `)`, must be escaped like usual in the terminal.

For example, this command will fail (silently) as crontab can't locate the file due to unescaped parentheses.
```
* * * * * /opt/anaconda3/bin/python some_path/to_my_(favorite)_python_script.py
```

This command will succeed as normal due to the escaped parentheses.
```
* * * * * /opt/anaconda3/bin/python some_path/to_my_\(favorite\)_python_script.py
```


##### Executing Python
Python follows the same rule of thumb -- to provide the full path to your Python installation as the command to run a Python script in your crontab.

In order to find the location of your Python install that's being used in your default environment, use the following commands in terminal:
+ `which python`
+ `which python3` (for Python 3 specifically -- might not need to be used if Py3 is your default)

These will give you outputs similar to:
+ `/opt/anaconda3/bin/python` (late 2020 distribution changed root path)
+ `/Users/<username>/anaconda3/bin/python`
+ `/Users/<username>/anaconda3/bin/python3`

Thus we would use these as our `python` command in our crontab as follows:

```
* * * * * /Users/<username>/anaconda3/bin/python my_python_script.py
```
or ...

```
* * * * * /opt/anaconda3/bin/python my_python_script.py
```

If you need to execute a Python script from a certain environment, you can simply specify that environment as your Python executable location, where `<env_name>` is the name of your desired environment.

```
* * * * * /opt/anaconda3/envs/<env_name>/bin/python my_python_script.py
```


##### Environment Variables in Cron
From [this article](https://serverfault.com/questions/337631/crontab-execution-doesnt-have-the-same-environment-variables-as-executing-user)


Cron always runs with a mostly empty environment. HOME, LOGNAME, and SHELL are set; and a very limited PATH. It is therefore advisable to use complete paths to executables, and export any variables you need in your script when using cron.

There are several approaches you can use to set your environment variables in cron, but they all amount to setting it in your script.

+ __Approach 1:__  
    Set each variable you need manually in your script.

+ __Approach 2:__  
    Source your profile:  
    `. $HOME/.bash_profile` (or `. $HOME/.profile`)    
    (You will usually find that the above file will source other files (e.g. `~/.bashrc` --> /`etc/bashrc` --> `/etc/profile.d/*`) - if not, you can source those as well.)

    This means to prefix your commands in the crontab with this sourcing code (note the semicolon `;`).  
    Ex: `* * * * * . $HOME/.bash_profile; /opt/anaconda3/bin/python my_python_script.py`

+ __Approach 3:__
    Save your environment variables to a file (run as the desired user):  
    `env > /path/to/my_env.sh`  

    Then import via your cron script:  
    `env - cat /path/to/my_env.sh /bin/sh`

+ __Approach 4:__  
    In some cases, you can set global cron variables in /etc/default/cron. There is an element of risk to this however, as these will be set for all cron jobs.

Comments:
+ _Approach 2_ is best for servers where you have sensitive things that shouldn't hit disk in env vars -- passwords, api keys, etc. Worked wonders for me, so thanks. – a p Nov 2 '17 at 21:57
+ _Approach 4_ worked for me in docker environment where the docker engine sets env vars. For this to work though, I had to save my env in the docker entrypoint. The complete solution is [here](github.com/rayyanqcri/swarm-scheduler) – hammady Nov 3 '17 at 5:02



Here's [another post](https://stackoverflow.com/questions/2229825/where-can-i-set-environment-variables-that-crontab-will-use) with some similar answers.



<BR>
<BR>

## Common Crontab Problems
This will not be an exhaustive list.  The four major issues I've encountered are as follows.
1. PATH problems (executables)
2. Unescaped characters in executable's file path
3. User access issue (change permissions or needs to be run from root)
4. MacOS specific: must add `cron` to the list of "Full Disk Access" processes. (MacOS 12 or higher, I believe)

<BR>

#### PATH Problems
Online forums discussing this abound.  Goes like this:  
Crontab is executed from a different environment than your default one.  As such, the `PATH` environment variable might be empty or incorrect (meaning the cron job cannot find where to find your installs, including Python, that would be found by default in your normal environment).  There are two ways to address this.

1. `export` your `PATH` env variable directly into the shell script you're calling in the cron job.  If you're not calling a shell script and instead are calling a different script directly, such as Python, then you'd either have to manage to import the `PATH` into the Python script itself (uncommon, unsure of validity), or import it into a wrapper shell script that then also calls the Python script in question once the `PATH` has been imported.

###### Inside the Shell Script
```
export PATH=/Users/jpw/anaconda3/bin:/usr/bin  ...etc.
```

2. "Source" your cron job file with your user environment profile, such as your hidden `.bashrc` or `.bash_profile` files.  This gives you access to _all_ your standard environment variables and not merely the `PATH`.  If you're wanting to run a script of some non-shell application, such as PHP or Python, you'll need to make a wrapper shell script which contains the following source line followed by the terminal command and accompanying script to run.  

    The key takeaway here is that the Python line would normally be an entry in our crontab, but since we are sourcing from our bash profile (to import our environment variables for our script to execute in), we made this wrapper shell script which now contains the Python script execution line that was previously a crontab line.  

###### Inside Your Wrapper Shell Script
Let's pretend we now have a wrapper shell script called `py_wrapper.sh`, which we use for sourcing from our default user profile and then running a Python script.  Note the shebang at the top of the script, because it is being executed by the `bash` command in crontab (see below).  

The contents of `py_wrapper.sh`:
```
#!/bin/bash
. ~/.bash_profile                               
/Users/<username>/anaconda3/bin/python my_python_script.py
```

In our crontab, the entry would look something like this (use `crontab -l` to view):
```
* * * * * /bin/bash /path/to/py_wrapper.sh
```

This job now executes by calling the `py_wrapper.sh` script, loading our environment variables from our bash profile, and then running the Python script.  Note we did not specify any time interval in our crontab, so this Python script would run every minute, every hour, every day, etc....

<BR>

#### Unescaped Characters in File Path
See the [short section above](#escaping-special-characters-in-file-paths) about following standard Terminal file path conventions.


<BR>

#### User Access - Change Permissions
Sometimes the permissions need to be changed on the script file itself in order for it to be executable.  I am unsure if the environment impacts this (don't think so), but I believe for a file to be called by the cron job command list, it has to be executable (I could be wrong).

Use this command in the terminal to make the target file, in this case a shell script, executable for the cron job:
```
chmod +x /path/to/your_file.sh
```

#### User Access - Root
Depending on the contents of the script to be run as a cron job, root level access might be required.  I'd argue that, in general, using a user-level crontab is best when viable.  However, if root access is needed, here's how you do it.

First, there are actually multiple crontabs that exist on your system -- one for the system-wide admin, and one for each separate user.  They do not overlap in their contents (though I believe both will execute if there is one for the active user, since the root one should always execute for all users).

The syntax and operation is the same as user crontabs.

To access and edit the root level crontab, enter `sudo crontab -e`.  
`sudo crontab -l` displays the root level jobs.

<BR>

#### MacOS App Permissions
In recent versions of MacOS (starting around 2017 or so), Apple implemented a more secure system-wide environment which requires the user to grant system-level access to any app requesting root access, as opposed to the opposite (which used to be the default).  Most fully baked commercial apps know this and alert you, the user, with a dialog box asking you to grant them system-level access.  They also usually show you how to do so (it's easy because these apps tends to be easily located in your `/Users/<username>/Applications` or `/System/Applications`, but still helps remove uncertainty).

But since cron is not such a commercial app (it's a core utility bundled with Unix-style operating systems), it does _not_ notify you that it needs this access, nor does it tell you how to grant it.  And this can be a problem since cron is not located in your `/Applications` folder.

Speaking personally, I had everything else set up for my crontab and it still would not run.  In the captured output (discussed above), the message I received was "..._operation denied._" (You might get some variation of this). So, if you're also receiving this message on a Mac system, then it is possible this is also your issue.  

The reason for this is based on whether any part of the cron job is running or reading from or writing a "protected" location as defined by MacOS.  Default user-facing folders such as `Desktop`, `Downloads`, `Documents`, etc. are all _protected_.  User-made folders, or certain hidden/system folders, such as `~/bin` or `~/log`, are _not_ protected and thus allow a cron job to execute without changing MacOS permissions.  (This is why the default cron logs in `~/log` don't trigger an error).

Here's an excerpt about debugging these errors with the __Console.app__ from an article in the Reference section:

>You can check this kind of errors by opening Console.app and searching for the shebanged exec in the script (here bash):
>
>```error   15:19:00.369105+0100    kernel  Sandbox: bash(4556) System Policy: deny(1) file-write-data /Users/user/Desktop/test/cronjob2.log```  
>
>
>```error   15:19:00.379093+0100    kernel  Sandbox: bash(4555) System Policy: deny(1) file-read-data /Users/user/Desktop/cronjob.sh```  
>
>
>In the examples above cron wasn't added to the Full Disk Access group.
>
>cronjob2 was run from an unprotected folder ~/bin but tries to write the log file to the protected folder ~/Desktop/test/. So no read error but a write error.
>
>cronjob was run from a protected folder ~/Desktop and tries to write the log file to the protected folder ~/Desktop/. So a read error.

Here's how to give cron the blessed status it so desperately needs (MacOS 10.15 - Catalina):
1. Click the Apple icon in the top left of your screen, in the menu bar.
2. Open _Security and Privacy_
3. Choose _Full Disk Access_ in the left sidebar
4. Click the `+` button beneath the window of privileged apps to add cron to the favored child list.  A Finder window will open, wanting you to pick the file / app to add.    
    _Note:_ You might have to unlock this preference pane by clicking the big golden padlock in the bottom left of the window and entering an admin password before you're allowed to add a new item to the system-wide access list.
5. Finding `cron` is difficult, so here's the shortcut: In this Finder window, hold down `CMD` + `SHIFT` and hit `G`.  This is the Finder shortcut for "go to." (You can also hit `CMD` + `SHIFT` + `.` to make all hidden items in the Finder appear.  This would allow you to navigate to the right directory in \#6 below.  Hit it again to toggle off the hidden items).
6. Enter this in the new window: `/usr/sbin/cron`
7. You're taken directly to the cron executable.  Select it.  Add it.

Now you should be set!


<BR>
<BR>


## References
+ Get help generating Crontab syntax at this [Cron Job-specific site](https://crontab-generator.org/) (very good)

+ A robust collection of [cronjob answers and tips](https://serverfault.com/questions/449651/why-is-my-crontab-not-working-and-how-can-i-troubleshoot-it) on Stack Overflow:

+ A great and [thorough overview](https://phoenixnap.com/kb/set-up-cron-job-linux)

+ [Python specific example](https://medium.com/@gavinwiener/how-to-schedule-a-python-script-cron-job-dea6cbf69f4e)

+ Some common cronjob [time intervals](https://phoenixnap.com/kb/set-up-cron-job-linux)

+ Here's a [short article](https://blog.shuaib.org/setting-up-cron-jobs-to-run-bash-scripts/) on different considerations such as enabling the root user

+ Some good alternative code snippets for cron at [Jessica's blog](https://www.jessicayung.com/automate-running-a-script-using-crontab/)

+ Really helpful [Wiki article](https://en.wikipedia.org/wiki/Cron)

+ A good [Stack Exchange](https://apple.stackexchange.com/questions/298775/cron-and-command-not-found) article on the PATH variable and issues with it.

+ Post on [adding MacOS permissions](https://apple.stackexchange.com/questions/378553/crontab-operation-not-permitted)

+ Another [really great post](https://serverfault.com/questions/449651/why-is-my-crontab-not-working-and-how-can-i-troubleshoot-it) with an answer about many details of cron, including logging and a GREP command to see if cron is even running.
