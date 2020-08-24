Unknown markup type 10 { type: [33m10[39m, start: [33m83[39m, end: [33m93[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m80[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m89[39m, end: [33m99[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m122[39m, end: [33m168[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m114[39m, end: [33m131[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m112[39m, end: [33m114[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m75[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m87[39m, end: [33m92[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m45[39m, end: [33m57[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m154[39m, end: [33m173[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m204[39m, end: [33m216[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m349[39m, end: [33m362[39m }

# Creating your own local YUM repository



In my current role, a point has come where I need to create RPM packages (I’ll cover this in a seperate article) and add them to our own private & local YUM repository, In order for us to install company approved and checked plugins and tools for use in production.

While this may create some toil initially, it’s a process that I have been able to ‘ansiblize’ fairly easily and now results in a fast, secure & vetted installation of items in our production environment (Websites).

## Tools required

In order to create your own local YUM repository you will simply need the YUM tool createrepo.

(This guide is for Linux distros and was tested on CentOS 7)

## Create directory to turn into repo

For this stage I just created the file structure of ~/repos/CentOS/6/5/Packages for ease but you can have anything you want aslong as it ends in Packages.

You will need to put at least one package in the /Packages dir, So I used a simple RPM that I have made.

![You can use the ‘tree’ tool to get a similar output to this in your terminal!](https://cdn-images-1.medium.com/max/4088/1*7FXsNTlYtT_35pLC_O5v7w.png)*You can use the ‘tree’ tool to get a similar output to this in your terminal!*

## Initialising The Repo

Now you have created the dir and added a package, you will need to initialise the repo, this creates the databases needed to handle all the package infomation, which allows the yum repo to serve the clients with speed and efficiency.

createrepo /repos/CentOS/6/5/Packages

Ensure that you initialise the /Packages dir as this will contain… the packages.

## Updating The Repo

If you wish you add new packages to the repo, you will need to update the repo using the createrepo tool with the command createrepo --update /repos/CentOS/6/5/Packages

This may also require a use of the yum clean all command at the client side.

## Publishing The Repo

To serve the Repo over HTTP using Apache it is very simple.
if you don’t have apache then you can install it with yum install httpd.

Once installed you will want to create a symlink between the Apache root and the root of our new repo using the ln utility.

ln -s /repos/CentOS /var/www/html/CentOS

Then start or retart the httpd service.

You should now be able to navigate to http://yourserver/CentOS/6/5/Packagesand see your packages.

![Here you can see my two RPM packages along with the repodata created when I initilised the repo.](https://cdn-images-1.medium.com/max/2000/1*pLfG89x19y5D-If-SdsgZQ.png)*Here you can see my two RPM packages along with the repodata created when I initilised the repo.*

## Configuring the Repo on a Client Machine

This stage is very straight forward and only requires the creation of one file.

Navigate to /etc/yum.repos.d directory and create a file called local.repo or anything .repoto be honest.

In the file you will need to put the name of the repo, and the URL of its Package Dir as so…

![Above you can see the very simple .repo file needed.](https://cdn-images-1.medium.com/max/4092/1*TqUYZZsrhdQzuifap-QAqw.png)*Above you can see the very simple .repo file needed.*

In the above file I have named the file mylocalrep stated its base URL, and enabled it (0 is no 1 is yes).

Now if everything is correct and you execute yum repolist on the client machine you should see the following -

![You can see my standard repos along with me new localrepo.](https://cdn-images-1.medium.com/max/4088/1*jc9FxAz3Rq6Pey26tPLpNA.png)*You can see my standard repos along with me new localrepo.*

If you use the command yum repo-pkgs localrepo list you can list the available packages in the repo.

![Now you can see my two new RPMs on my new local YUM server!](https://cdn-images-1.medium.com/max/4096/1*J5GDy1xXOnR1QbPuj_n33g.png)*Now you can see my two new RPMs on my new local YUM server!*

Hopefully this guide has helped you, the setup is actually pretty straight forward, the only issue is when you add a new RPM you will need to remember to createrepo --update your new repo, and if you yum yum repolist on your client and the status hasent changed (the status number shows the amount of packages in the repo) then you will need to run yum clean all on the client.

If you run into any issues feel free to reach out to me!

*Cover Photo by [Arian Darvishi](https://unsplash.com/@arianismmm?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/developer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*
