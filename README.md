This simple cfc allows you to set up an one-button deployment process that uploads your code straight from GitHub onto your server. The current setup supports only ColdFusion on Windows server with IIS. You are free to use this code to create solution for other platforms.

### Installation

The main idea of the approach is to let native Git client to do his work. Each site installation should be a cloned git repository in order to let git client manage the directory.

1.	You need to Install [Git for Window](https://git-for-windows.github.io/) on each of your web servers. Please note, you may want to make sure that during the installation wizard you have a line ending option selected to *“Checkout as-is, commit as-is”*, it is not by default.

2.	To generate a new ssh key run a small program called git-gui.exe (`c:\Program Files\Git\cmd\git-gui.exe`).
There is an option under *"Help"* menu called *"Show SSH Key"*, when you click *"Generate Key"* button it creates two files in `C:\Users\<your_username>\.ssh\` folder:  
   **id_rsa** - secret key  
   **id_rsa.pub** - public key

3.	Add the public key to a github account that will be used on web servers to access repositories, separate account with read-only access for certain repositories is the most secure way. (`Github.com > Settings > SSH keys > New SSH key`).

4.	Also you will probably need to create a file *".profile"* in user's home folder with the following text:  
   `GIT_SSH="/usr/bin/ssh.exe"`

5.	Now you can test the setup, open windows console (cmd.exe) and try to clone a repository:
`c:\Program Files\Git\cmd\git.exe clone --branch master --progress -v "git@github.com:acnot/cf-git.git" "C:\cf-git"`

6.	Copy *“.ssh”* folder and *“.profile”* file to home directory of the windows user that runs *Coldfusion Application* server, on each of your web servers.

7.	The next thing is that you need to put *gitbot.cfc* somewhere, it should be the same url path on each server, by default it is a root directory. If you put the cfc in `C:\inetpub\wwwroot\` it should be accessable by `http://<local_server_id>/gitbot.cfc`. If this is not working in your IIS setup, then create new site in IIS and add that domain name to windows hosts file on main server where you will run the gitbot.

8.	Git doesn’t allow to clone repositories into a folder with some files, so just clone your repository somewhere else and then copy *“.git”* folder to your site directory.

9.	Before first running of the gitbot make sure your sites don’t have uncommitted changes, otherwise it will be removed. Also create *“.gitignore”* file in the root repository folder and list there all folders and files that should be on the servers but not in the repository, for example git ignore list could include:
```
*.ffs_db
*.DS_Store
Thumbs.db
backup/
```

### Configuring

The following list of settings can be found in the *gitbot.cfc*:

**master_server** — an array with domain names for the main server where you will be using the gitbot, it can be string domain names and IP addresses

**script_location** — url path to the gitbot, the same on each of your servers

**credentials** — username and password for basic authentication to access the page

**sites** — a struct with details for your sites, struct key is an internal id which also used for sorting in the drop-down, struct value is another struct with details for a site:

**name** — internal name, if contains a word *“prod”* the option in the drop-down will be red

**host** — domain name of a site which displayed in the drop-down

**servers** — an array of arrays, the first element of each array is server domain name (can be ip address), the second is physical path on the server to site directory

**branch** — default branch for a site, will be populated into form when this site is selected

### Usage

The URL of the page is `http://<your_main_server>/gitbot.cfc?method=page`

A simple html form allows you to select site and branch which you want to upload into the site directory. Optionally you can specify commit hash or tag name, useful when you want quickly revert unwanted commits on production and take some time to prepare a fix.

The hardcoded git commands will discard any local changes for versioned files on your site, unversioned files will stay.

In the page output you can see all git commands the bot is running and its result. The last two command are for your control, they display current commit message and author, list of unversioned files (add files you don’t want to see here in the *.gitignore* file).
