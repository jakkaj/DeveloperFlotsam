

Tech: Node.js, Visual Studio Code, Nuget, Azure App Service Site Extensions, Kudu

[Funcgraph on GitHub](https://github.com/jakkaj/funcgraph) (including install instructions)

*This is a light overview on how the Azure Function Grapher SiteExtension was built.*

Azure Functions are great - you can quickly build software to solve some advanced scenarios using event based serverless code. The whole idea is that you write small "functions" that each solve their little piece of the puzzle.

Functions can take input from many sources and events like http, service bus, table writes, file writes etc. They can also output to these to in turn call more functions and other services.

The trouble is that as your function projects grow, as you add more and more input types it can be hard to see what is calling what, via which event/service etc.

<img src="https://cloud.githubusercontent.com/assets/5225782/24825002/35c2318c-1c59-11e7-9c9c-155ce0e14267.png" width="720" />

Last week I saw a blog post Mathias Brandewinder (@brandewinder) <a href="http://brandewinder.com/2017/04/01/azure-function-app-diagram">here</a> and it got me to thinking how cool it would be if that would work on my function sites.

Mathias' idea is that you iterate through functions on the file system, collate their settings from the function.json files before creating a visual graph.

This lead me to think about creating an Azure Site Extension. How hard could that be?

The great thing about Site Extensions is that you can install them in any (non-consumption plan based) function site and they will just start working. Even better is there is a really nice site (<a href="https://www.siteextensions.net/">https://www.siteextensions.net/</a>) that you can use to search and install them for you.
<h3>Project Requirements</h3>
<ul>
 	<li>It's a SiteExtenion</li>
 	<li>Iterates function.json files to gain data on how they are triggered and what outputs they send</li>
 	<li>Generates GraphWiz graphs</li>
</ul>
<h3>Node Environment Setup</h3>

<a href="http://char.gd/microsoft/setting-up-perfect-windows-dev/">This guide</a> is a great example of a dev environment and is close to my environment. I use the Windows Subsystem for Linux (WSL) and Visual Studio Code (with Bash shell as terminal prompt).

I decided to build the new SiteExtension in Node. Node works well in the Azure App Service (i.e. inside Kudu) and there is an NPM package that allows the generation of GraphViz graphs.

Visual Studio Code is a great editor for <a href="https://code.visualstudio.com/docs/nodejs/nodejs-tutorial">Node development</a> with code highlighting and great debugging capabilities.

This project is using ES6 and makes use of classes and arrow functions - they make JavaScript so nice to work with :)
<h3>Azure App Service File Layout</h3>
Along side the code of an Azure Function is a configuration file called function.json that this project queries to see the input and outputs. All functions are run inside the Azure App Service which is a custom version of IIS running on top of Windows.

You can log in to and browse the site files using a tool called Kudu which is installed in all App Service sites (https://&lt;yoursite&gt;.scm.azurewebsites.net/). Using this I was able to discover the Functions sitting in D:\home\site\wwwroot.

<img src="https://cloud.githubusercontent.com/assets/5225782/25035785/fa784290-2132-11e7-92f3-b9b734953b5e.PNG" alt="kudu" width="720" class="aligncenter" />

<h3>GraphWiz</h3>
I started off by seeing if I could even generate GraphWiz graphs in node. Sure enough there is an npm package called <a href="https://github.com/mdaines/viz.js/">viz.js</a> that does the job nicely.


<script src="https://gist.github.com/jakkaj/2cc7a86dab64f4112efd079de5cc9adc.js"></script>

(<a href="https://github.com/jakkaj/GraphVizNodePlay/blob/master/basictest/test2.js">Full source on GitHub</a>)

There is also a dependency on svg2png. Viz.js is able to output PNG format but not in Node, it requires the browser context. Svg2png is able to achive this in Node by using the phantomjs headless browser.

<h3>Iterate function.json files</h3>

When running inside Azure (e.g. as a SiteExtension) you have access to all App Service config variables on ```process.env```. One of these variables ```process.env.HOME``` contains the full path to the site root (i.e. where the Functions are stored).

The project needs to be able to "walk" the folders under the site root of the Azure Site to discover the function.json files. There is an npm package called [walk](https://www.npmjs.com/package/walk) for just this. 

<a href="https://github.com/jakkaj/funcgraph/blob/master/SiteExtension/funcgraph/funcwalk.js#L152">This code</a> demonstrates walk in action.

### Dot Files

GraphWiz uses a format called Dot to generate graphs. It's a bit like a strange version of Json. 

[dotgraph.js](https://github.com/jakkaj/funcgraph/blob/master/SiteExtension/funcgraph/dotgraph.js) is a little helper class to assist with generation of the dot formatted text. 

<script src="https://gist.github.com/jakkaj/bc5974024cd28cdf733414aefaf17ca0.js"></script>

This code takes the consolodated data from the function.json files and produces the dotgraph format ready to rendering in to a graph. 

Building the graph is super simple using viz.js.

<script src="https://gist.github.com/jakkaj/a31254ff0da75f3f2de51b32d2752e08.js"></script>

Once that is all generated, the result is returned to the code in [server.js](https://github.com/jakkaj/funcgraph/blob/master/SiteExtension/funcgraph/server.js) for delivery to the user. 

### Site Extension

Creating a site extension is super easy. 

1. Add a [web.config](https://github.com/jakkaj/funcgraph/blob/master/SiteExtension/funcgraph/applicationHost.xdt) file to the project so IIS knows how to work with the node project. 
2. Add a [applicationHost.xdt](https://github.com/jakkaj/funcgraph/blob/master/SiteExtension/funcgraph/applicationHost.xdt) file which allows the App Service to include this site as part of the sites offered on Kudu. 
3. Copy the site up to d:\home\SiteExtensions\funcgraph.
4. Click Restart on the SiteExtensions tab. 

This is enough to get the site extension going. It's easy because site extensions are just regular App Service sites with some extra config (applicationHost.xdt) and living in a differnt home (not wwwroot, but SiteExtensions).

You can now access the grapher at https://yoursite.scm.azurewebsites.net/functiongraph. 

### Make available on https://www.siteextensions.net/

Site extenions are nuget packages - but instead of being hosted on Nuget.org, they are hosted on siteextensions.net. If you've used nuget.org before then siteextensions.net will seem very familiar. 

The [nuspec](https://github.com/jakkaj/funcgraph/blob/master/funcgraph.nuspec) generates a simple nuget package ready for upload using the ```nuget pack``` command. Install nuget from [here](https://dist.nuget.org/index.html) and make sure it's in your OS path. 