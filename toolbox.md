---
layout: page
title: My Data Science Toolbox
permalink: /toolbox/
---

<style>
// Using numbers instead of bullets for listing
#markdown-toc ul {
    list-style: decimal;
}

#markdown-toc {
    border: 1px solid #aaa;
    padding: 1.5em;
    list-style: decimal;
    display: inline-block;
}
</style>

* Table of Contents
{:toc}
  
<p>_last updated Sep 25, 2017_</p>
---  
  
# Philosophy  
I try to follow a simple set of guidelines when approaching my workflow.
 - __Use the right tool for the job.__ R is wonderful, but if all I need is a simple chi-squared test, I'm going to use jamovi. If I need to do some heavy scripting to assign stimuli to lists, I'm going to use python. If there's a toolbox that does what I need to do in Matlab, I'm going to use Matlab. Comfort shouldn't dictate the approach.  
 - __Make good habits easy.__ Take the time to arrange my workstation so that the things I know I "should" do are easy and salient.
 - __Make it modular.__ I re-use code all the time, because I work with similar datasets that pose similar problems. I also want it to be easy to swap out some process with some other process mid-stream (e.g. switch my outlier detection method).
 - __Make it reproducible.__ Script everything, use version control, make regular backups, keep it organized, write documentation, comment your code, and make all of these habits easy.
 - __Make it accessible.__ Where possible, use free tools, support open-source projects, share data and materials, and host non-paywalled author versions of manuscripts.

The following recommendations will focus on Windows programs that are available for free, though I try to emphasize those that also have Linux and Mac versions.  
  
# The Toolbox Core
My workflow revolves around three primary languages: R, Python, and Matlab. 
