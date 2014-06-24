---
layout: post
title: "Mercurial Branching"
date: 2012-07-11 00:08
comments: true
categories: 
---
We are currently going down the path of switching from our huge monolithic SVN repository, to a number of product based Mercurial repositories. Historically, I have used FogBugz and Kiln, but this time we have decided to give the JIRA stack a whirl - mainly for GrassHopper, as one of the project managers here wanted to give it a go.

In the old SVN repo, instead of branching and tagging, we copied files to "Release folders", and ended up with a bunch of duplicated code, which was pretty hard to manage. One problem was making a bug fix in a release directory, then having to either manually implement that into the development directory, or do an onerous merge of the 2 directories.

To cut a long story short... We were using SVN like a file store, not a source control system.

For my personal projects in KilnHg I have always had 2 repositories per project - one for "Release" and one for "Development". Why? Because that is how Fog Creek's guides showed me. It works, but I wanted to take advantage of Mercurial's named branches for our work projects. It produces some nice graphics, and (whilst I prefer the terminal to work with hg) works better with the TortoiseHg GUI - which is what my developers are used to, albeit TortoiseSVN.

So, I armed myself with some questions:
<ul>
	<li>How to do this?</li>
	<li>What is the best way?</li>
	<li>Am I mad?</li>
</ul>
And went to the holy grail of Google to have a look.

After some to-ing and fro-ing, trying and failing, I came up with the following "demo" scenario. What follows is simple, and possibly does not deserve a blog post - but it can provide some point of reference for me (and my team!)...
<h1>Initial repository</h1>
<ol>
	<li>Create repo in source control system</li>
	<li>Clone repo to my machine
<pre>hg clone repoURL</pre>
</li>
	<li>Add my code to the repo
<pre>hg add</pre>
</li>
	<li>Commit code to the repo
<pre>hg commit -m "Initial commit"</pre>
</li>
	<li>Push to the remote repo
<pre>hg push</pre>
</li>
</ol>
<h1>Make the release branch</h1>
<ol>
	<li>Make the release branch
<pre>hg branch "Release-1.0"</pre>
</li>
	<li>Commit the change to the repo
<pre>hg commit -m "Added branch Release-1.0"</pre>
</li>
	<li>Push to the remote repo
<pre>hg push --new-branch</pre>
</li>
</ol>
<h1>Bug fix on the Release-1.0 branch</h1>
<ol>
	<li>Switch working directory to the branch you want to edit in your repo
<pre>hg update "Release-1.0"</pre>
</li>
	<li>Add your bugfix to the repository</li>
	<li>Add your changes to the repo
<pre>hg add</pre>
</li>
	<li>Commit your changes
<pre>hg commit -m "Added BUGFIX1 to Release-1.0"</pre>
</li>
	<li>Push
<pre>hg push</pre>
</li>
</ol>
<h1>Feature added on the default branch</h1>
<ol>
	<li>Switch working directory to the default branch
<pre>hg update "default"</pre>
</li>
	<li>Add your feature to the repo</li>
	<li>Add your changes to the repo
<pre>hg add</pre>
</li>
	<li>Commit your changes
<pre>hg commit -m "Added FEATURE1 to default"</pre>
</li>
	<li>Push
<pre>hg push</pre>
</li>
</ol>
<h1>Merge the bugfixes on Release-1.0 into default</h1>
<ol>
	<li>Switch working directory to the default branch
<pre>hg update "default"</pre>
</li>
	<li>Merge the Release-1.0 branch into the default branch
<pre>hg merge "Release-1.0"</pre>
</li>
	<li>Commit your changes
<pre>hg commit -m "Merged Release-1.0 into default"</pre>
</li>
	<li>Push
<pre>hg push</pre>
</li>
</ol>
Which results in the following graphlog (from BitBucket):

![](http://i.imgur.com/wyhXO.png)