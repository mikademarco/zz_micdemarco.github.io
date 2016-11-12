---
author: micdemarco
comments: true
date: 2015-06-25 15:59:07+00:00
layout: post
link: https://micdemarco.wordpress.com/2015/06/25/mvc-angular-less-gulp-tfs-vso/
slug: mvc-angular-less-gulp-tfs-vso
title: MVC + Angular + Less + Gulp + TFS + VSO
wordpress_id: 41
tags:
- angular
- Gulp
- MVC
- TFS
---

[![asp.net-gulp](/assets/2015-06-26_02-04-09.png)](/assets/2015-06-26_02-04-09.png)

The MVC [Bundler ](http://www.asp.net/mvc/overview/performance/bundling-and-minification)is a great tool provided by Microsoft and great for bundling and optimizing sources for delivery to the browser.  However as the front end content and libraries grow, it starts to lack the capabilities needed to effectively build the front end files both at run-time and also during development.

What if you wanted to build less files, minify and combine your scripts as you are developing them?  What if you wanted to call "bower install" as part of your build?

The in built MVC Bundler starts to feel limited when your front end build increases in complexity.

In this blog post I am going to describe the steps that I took in order to:
	
  1. use bower to install front end packages

	
  2. use gulp to watch and compile my less files every time I make a change

	
  3. use gulp-bundle-assets to compile source files every time I make a change

	
  4. build all front end scripts and styles into separate files for vendor and application

	
  5. use the MVC Bundler to serve the compiled files

	
  6. integrate the front end build with an on premises TFS Build connected to VSO TFS


Throughout the article I will reference the following:

Solution root: C:\Code\Demo\
MVC project root C:\Code\Demo\Demo.WebUI


# 1. Use bower to install front end packages


Firstly set up your environment.  (note these steps are for VS2013 and can be skipped if you are using 2015)



	
  1. Read [http://www.hanselman.com/blog/IntroducingGulpGruntBowerAndNpmSupportForVisualStudio.aspx](http://www.hanselman.com/blog/IntroducingGulpGruntBowerAndNpmSupportForVisualStudio.aspx)

	
  2. Install Git (required by bower) : [https://git-scm.com/downloads](https://git-scm.com/downloads) (add Git commands to the path)
[![Git](/assets/2015-06-25-22-22-23.png)](/assets/2015-06-25-22-22-23.png)

	
  3. Install Node : [https://nodejs.org/download/](https://nodejs.org/download/)

	
  4. Install Task Runner: [https://visualstudiogallery.msdn.microsoft.com/8e1b4368-4afb-467a-bc13-9650572db708](https://visualstudiogallery.msdn.microsoft.com/8e1b4368-4afb-467a-bc13-9650572db708)

	
  5. Optional install Alt+Space Command Line shortcut: [https://visualstudiogallery.msdn.microsoft.com/4e84e2cf-2d6b-472a-b1e2-b84932511379](https://visualstudiogallery.msdn.microsoft.com/4e84e2cf-2d6b-472a-b1e2-b84932511379)
This is useful to quickly open a shell and run npm/gulp commands

	
  6. Open cmd and install global commands:
>npm install -g gulp
>npm install -g bower


Next we need to add front end packages to the project using bower.


### Option 1 - Easy way


1. Add a file bower.json to the project root

{% gist micdemarco/8469cb120eb20e1313b4 %}

2. Add a file packages.json to the project root

{% gist micdemarco/54d241a4a33050893f21 %}

3. Open a command prompt and run


C:\Code\Demo\Demo.WebUI>npm install
C:\Code\Demo\Demo.WebUI>bower install





### **Option 2 - Ninja way:**





	
  1. Open a command prompt in the MVC project directory

	
  2. Init npm
C:\Code\Demo\Demo.WebUI>npm init
...

	
  3. Init bower
C:\Code\Demo\Demo.WebUI>bower init
...

	
  4. Install packages using bower install --save
C:\Code\Demo\Demo.WebUI>bower install angular --save
...

	
  5. Install npm packages using npm install --save
C:\Code\Demo\Demo.WebUI>npm install gulp-less --save
...




### After that...


The result of the above steps is a directory structure like the following:

[![bowercomponents](/assets/2015-06-25_22-48-19.png)](/assets/2015-06-25_22-48-19.png)

Great, we are now managing all front end libraries with bower.  These can be referenced by your cshtml files as:

```html
<script src="/bower_components/angular/angular.js"></script>
```

# 2. Use gulp to watch and compile my less files every time I make a change


One of the best uses of having a task runner is the ability to watch files and perform transformations with a few lines of code.

A major application is for the compilation of less files.  I would like to:



	
  1. Split my less files by category

	
  2. Use bootstrap variables inside my less

	
  3. Compile all my less files into a single css file


Approach

1. Create a master style.less that points to all application less files:

{% gist micdemarco/b18032252bc51943fc66 %}

2. Create a gulp task to watch you less files, and compile the less:

{% gist micdemarco/3e083b51ecb83e4059b4 %}

Note: Line 1 tells the Task Runner plugin to run the task.  This is slightly different in VS 2015
/// <vs SolutionOpened='dev:watch' />

All that is needed is to reference the compiled css:

```html
<link href="/Content/Css/style.css" rel="stylesheet" />
```

# 3. Use gulp-bundle-assets to compile source files every time I make a change


I would also like to compile my JavaScript files whenever I make a change.  In order to do this, I need to extend my watch task to also watch my .js files.  I also need to use some new tools to perform the compilation

Approach:

1. Recommended reading



	
  1. [https://github.com/johnpapa/angular-styleguide](https://github.com/johnpapa/angular-styleguide)

	
  2. [https://github.com/toddmotto/angularjs-styleguide](https://github.com/toddmotto/angularjs-styleguide)

	
  3. [https://scotch.io/tutorials/angularjs-best-practices-directory-structure](https://scotch.io/tutorials/angularjs-best-practices-directory-structure)


2. Implement a folder structure that allows you to

	
  1. Identify shared dependencies

	
  2. One object to one file

	
  3. Wrap in IIFEs

	
  4. Identify each file as a filter/service/directive/controller


3. Create a bundle file for use with: [https://www.npmjs.com/package/gulp-bundle-assets](https://www.npmjs.com/package/gulp-bundle-assets).

This file will go under
C:\Code\Demo\Demo.WebUI\Content\Bundles\bundle.main.copy.config

{% gist micdemarco/052f3e3c941a3caeb9a3 %}

4. Create a new task in the gulpfile.js to build the scripts and extend the watch task:

{% gist micdemarco/8268d10fd10bb423bf9b %}

**Note: the rev:false option will always generate the same output file.  **

The watch task will ensure that every change to a *.js file under /Content/App will generate a new scripts bundle.  It can be referenced as:

```html
<script src="/Content/Bundles/Debug/assets/main-scripts.js"></script>
```

# 4. Build all front end scripts and styles into separate files for vendor and application


I would now like to take my front end build a few steps farther.  I would like to



	
  1. build my vendor scripts and styles into single files

	
  2. add copy tasks for fonts, images and html templates

	
  3. add a task to clean the output directory before build

	
  4. optimise my watch tasks to only build and copy what is necessary

	
  5. call npm install and bower install as part of my build


1. Add a bundle config for main styles:

{% gist micdemarco/6785bc49ee9420aae0be %}

2. Add a bundle config for copying template files:

{% gist micdemarco/55c4709399dc948be15a %}

3. Add a bundle config for vendor files:

{% gist micdemarco/3d7dfac443ff882c3db2 %}

4. Add a bundle config for copying fonts and images

{% gist micdemarco/a8e6e748cb01b1917588 %}

5. Add gulp tasks for each of the separate bundle config files, and a main **bundle** task to run them all in sequence:

{% gist micdemarco/06661961beab095b9eca %}

**Note 1: Also added an install task to run "npm install" and "bower install" from gulp**
** Note 2: Added task runner configuration /// <vs BeforeBuild='bundle' SolutionOpened='dev:watch, install' />**

In order to reference the correct files I will need the following references:

```html
<link href="~/Content/Bundles/Debug/assets/vendor-styles.css" rel="stylesheet" />
<link href="~/Content/Bundles/Debug/assets/main-styles.css" rel="stylesheet" />

<script src="/Content/Bundles/Debug/assets/vendor-scripts.js"></script>
<script src="/Content/Bundles/Debug/assets/main-scripts.js"></script>
```

# 5. Use the MVC Bundler to serve the compiled front end files


The difficulty with integrating the gulp-bundle-assets file into MVC was how to generate unique files and reference them from the razor page.

gulp-bundle-assets can generate unique file names such as **vendor-scripts-d2e93d010x.js **by setting rev:true in the bundle config file.  Upon bundling, it will output the name of the created filenames to an output file called bundle.results*.js

However it is difficult to reference these files from an MVC Razor page.

The solution was to finally use the MVC bundler to pick up these files.

Approach:

1. Configure 2 bundles in BundleConfig.cs:

{% gist micdemarco/2d306d6917914087f600 %}

**Note: The BundleConfig can be configured to serve different bundles such as Release/Debug based on the environment.**

2. Update the razor to reference the MVC bundles instead of static files
@Styles.Render("~/bundles/styles")
@Scripts.Render("~/bundles/scripts")

Now we can be sure that the files served to the client will always be up to date and will not be stored in the browser's cache.

**Note: Through the use of source maps, you can also debug and edit the source files inside chrome by mapping the scripts to the source code directory within dev tools:**

[![chrome](/assets/2015-06-25-22-22-22.png)
](/assets/2015-06-25-22-22-22.png)**Figure: Adding mapped sources to file system from chrome dev tools**


# 6. Integrate the front end build with an on premises TFS Build connected to VSO TFS


Ok, now that we have done all this work, it is time to check in some code .... (what!! you haven't checked in anything yet?!)

The question is, whether to check in compiled files or not.  Wouldn't it be great if the build server could bundle the front-end files in the same way as on the development machine?

I would like to



	
  1. Configure my on premises TFS to perform the gulp bundle task as part of the build process

	
  2. Include the bundled files as part of the deployment


Approach

1. Read the excellent post on how to do this: [http://www.codecadwallader.com/2015/03/15/integrating-gulp-into-your-tfs-builds-and-web-deploy/](http://www.codecadwallader.com/2015/03/15/integrating-gulp-into-your-tfs-builds-and-web-deploy/)

2. On the build server perform the steps in **"1. Use bower to install front end packages"**

3. Ensure that the user that the build service is running under can execute


>bower
>gulp
>npm


4. Add the following sections to your csproj at the end after all the imports sections:

{% gist micdemarco/c20700926f98e5a2f0c8 %}

5. The only files that you need to check in are your gulpfile.js and /Content/Bundles/bundle.*.js



	
  * Everything else under /Content/Bundles/ can be ignored

	
  * Everything under /bower_components/ can be ignored

	
  * Everything under /node_modules/ can be ignored




# Conclusion:


With a bit of configuration, it is possible to integrate the packaging power of npm into visual studio, and enjoy the great flexibility that the gulp task runner provides.

There exist great tools such as gulp-bundle-assets that make the minification and bundling easy, however you could also perform these tasks using individual packages and have even more control on the build process.

MVC bundler is still a useful tool for the final step of delivery of the bundled files to the client.

With a few modifications, it is possible to integrate npm and gulp with MSBuild to fully automate the front end build process.

This allows you to keep your code base clean and not need to check in any compiled files into your source control.
