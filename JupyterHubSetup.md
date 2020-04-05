---
title: JupyterHub Setup
has_children: false
nav_order: 4
---

# Setting up a JupyterHub for Teaching Workshops on AWS

JupyterHub is a great way to run basic Python coding workshops. People can come in and start coding with zero setup ahead of time on their own machines--all they need is a web browser. This page describes my method of setting up a JupyterHub on Amazon Web Services, adding workshop data to the JupyterHub, and distributing lessons to the participants as Jupyter Notebooks via GitHub.
![Python Logo][Python] ![Pandas Logo][JupyterHub]  

***
Phil White, Earth Sciences Librarian, University of Colorado Boulder
[philip.white@colorado.edu](mailto:philip.white@colorado.edu)

***

Prerequisites:
-[Amazon Web Services](https://aws.amazon.com/) account
-[GitHub](https://github.com/) account & [basic understanding of GitHub](https://towardsdatascience.com/getting-started-with-git-and-github-6fcd0f2d4ac6)
-Some command line knowledge
-Some Python programming experience
***
## Part 1: Deploying JupyterHub

The easiest and probably most appropriate deployment of JupyterHub for workshops and smaller classes is [The Littlest Jupyter Hub (TLJH)](http://tljh.jupyter.org/en/latest/). TLJH is ideal for 100 users or less.

[Install TLJH on Amazon Web Services](http://tljh.jupyter.org/en/latest/install/amazon.html). I can do no better than TLJH's own very clear instructions. This is straightforward and can be accomplished in literally just a few minutes. I was successful in launching my own JupyterHub on an EC2 instance on the first try, and I had zero experience doing something like this at that time.

Be sure to save your key pair in a safe place.

I have only deployed TLJH on Amazon Web Services, but there are great instructions for [installing TLJH on other platforms](http://tljh.jupyter.org/en/latest/install/index.html) as well.

### Some things to be aware of:
- You'll be deploying TLJH on an EC2 instance.
- You can set up your Hub on EC2's free tier initially, which I recommend...
- BUT you will want to upgrade memory and processing power when you're actually running the workshop. See Part 4 below.
- AND a server costs money! But, with AWS, it only costs money while you're running it. And you may qualify for the free tier for one year. So, your initial installation will be set up with less processing power and less memory, qualifying for the free tier (for one year). This will be nice while you're learning your way around.
- When you're not using it, I recommend getting into the habit of shutting off. When/if you upgrade, you'll want to shut it off when not running so as to avoid running up your bill unnecessarily.
- All that said, AWS is very affordable. Here is a nice [breakdown of the pricing structure](https://www.ec2instances.info/).
- Per the instructions, I am running a t2 instance, because my initial set up with t2micro was free for the first year. I upgrade/downgrade only when necessary and keep it turned off when not using.
- Don't forget to turn off when not using! Otherwise you'll have a surprise bill (As a point of reference, I accidentally left a different EC2 instance running for about 2 weeks amounting to a monthly bill over $20). But this was a high powered instance. Most are pennies per hour.

## Part 2: Set up the JupyterHub  

In AWS, under Instances, you can launch/shutdown your instance (Actions>Instance State>Start). Then, your JupyterHub's url will be visible as IPv4 Public IP. Mine looks like this:


Copy and paste the URL into a new browser tab and visit your JupyterHub. Log-in with the username and password you set up previously (or set your password on first log-in).


If you've used Jupyter Notebooks before, this will look familiar. The only difference is that with Hub it's served on the web instead of serving from  your local computer.


#### Add all necessary Python packages

You'll need to add all of the Python packages that you'll be using in the workshop. The default installation comes with most everything you'll need, but likely you'll need to add a few.

Open up a terminal within JupyterHub:

You could SSH into your server using the key pair and an SSH client like PuTTY or something similar. But, it's easy and just as effective to use the terminal within JupyterHub.

As noted in the installation instructions, you'll need to use sudo commands along with your typical package manager (like pip or conda) to install the packages you need, like so:

```
sudo -E pip install geopandas
```
OR
```
sudo -E conda install geopandas
```

In my most recent GeoPandas workshop, I installed the following packages using these commands:
```
sudo -E pip install pandas
sudo -E pip install geopandas
sudo -E pip install matplotlib
sudo -E pip install descartes
sudo -E conda install rtree
```

Following my methods, you will also need to install Git on your instance:
```
sudo apt install git
```

#### Set up your data directory  

There are [different methods of both adding data to your JupyterHub](http://tljh.jupyter.org/en/latest/howto/content/add-data.html) and [distributing data to your users](http://tljh.jupyter.org/en/latest/howto/content/share-data.html#howto-content-share-data). One would be to store data on GitHub and have each individual download it and add it themselves. Another would be to add it to each user's folder. However, there are downsides to those methods: First, having participants add it themselves is one more thing that could fail and that's what we're trying to avoid here, right? Adding it to each person's user directory would require extra planning and hassle on your behalf.

<em>My preferred method</em> is to add all workshop data from a GitHub repository to a read-only folder that all users have access to. Then, during the workshop, they can all read the data and perform tasks without any problems because all of their individual operations take place in RAM; any output files can be written to their own directory.&ast;

A bonus is that by storing the data on GitHub, users can access it later when/if your server is inactive.

Another bonus is that this takes up much less space on your server instance. Instead of duplicating the workshop data on each users' space on the JupyterHub, they will each access the same folder.

In the end, this solution saves you time <em>AND</em> money.

Here are the steps:

1. Create a GitHub repository and add all of the workshop data to the repository. If you're new to GitHub, this is easy. Here is a [GitHub Guide](https://guides.github.com/activities/hello-world/) and [how to upload files](https://help.github.com/en/github/managing-files-in-a-repository/adding-a-file-to-a-repository).

2. On your JupyterHub, navigate to your root directory. In your JupyterHub's terminal, you'll first need to change directory back to the root directory. This should do it:
```
cd ../../
```
The working directory will now be jupyter-your_user_name@ip-your_ip_address:/. An ls command should show you the following directories:

<insert image>

3. Next, create a new folder in your data directory named 'workshopdata'. The data directory is located at /srv/data. From your root directory, use the following command:
```
sudo mkdir -p /srv/data/workshopdata
```

4. Next, add an alias 'workshopdata' directory to all users' folders. Essentially, this makes a read-only workshopdata folder in each user's directory. All new users will have a workshopdata folder when they log into their space on the JupyterHub. First, change directory to the etc/skel folder:
```
cd /etc/skel
```
Then, create the alias directory:
```
sudo mkdir -p /srv/data/workshopdata workshopdata
```

5. Now, navigate back to your 'workshopdata' folder. First, back out to your root directory:
```
cd ../../
```
Then:
```
cd /srv/data/workshopdata
```

6. Now, you will make your 'workshopdata' folder a Git repository:
```
sudo git init
```

7. Now you will pull all of your workshop data from the GitHub repository you created in step one into this workshopdata directory using Git commands. First, connect to your data repository:
```
sudo git remote add origin https://github.com/your_username/you_data_repo.git
```
Then pull the data:
```
sudo git pull origin master
```
This will add all of your workshop data on GitHub to your JupyterHub's workshopdata directory. If you update your data repository on GitHub later, just use the sudo git pull orgin master command to update your workshopdata directory.

Now, each new user will have a read-only data directory in their space on the JupyterHub.

&ast;Using this method underscores the importance of users being able to both navigate filesystems and change directories using Python commands. This means using the Python package 'os'. They'll need to change to their home directories at the beginning of each lesson and access each dataset using the workshopdata/ prefix. [See example here in codeblock 2 and 3](https://outpw.github.io/2.%20Selecting%20%26%20Filtering%20by%20Attributes%3B%20More%20Plotting.html)

#### Adding Users to the JupyterHub:
Adding users to the JupyterHub is extremely easy. Your choice is when to add them. You can add users in advance, which is helpful if you know you will have a large workshop. But because it is so easy to do, I typically create the users on the fly as workshop participants walk into the room. I will ask them what they want their username to be and then create it.

To create users, click Control Panel in your JupyterHub, then Admin.

Next click add users, then type in user names separated by a return, like this:

<insert image>

### Part 3: Distributing workshop Notebooks


### Part 4: Upgrading your EC2 instance


### Final Notes

- This is an awesome way to run a workshop. Because I take the time to prepare this way, I have never ran into technical issues and problems during the workshop. Each time I teach a workshop using this setup, participants praise the smoothness of the workshop. Everthing just works.

- I usually create a second user account for myself set up as a non-admin so I can test everything ahead of time to ensure things work as expected from the participant's perspective. I also use this non-admin user account during the workshop, and delete the notebooks folder so I walk through the exact same steps as each participant, and we all see the same things happening.

[Python]: img/PythonLogo.png
[JupyterHub]: img/hublogo.png
