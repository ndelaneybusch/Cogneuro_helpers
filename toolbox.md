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
  
<p><i>last updated Sep 25, 2017</i></p>
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
  
## R Setup
Nearly all of my explanatory modeling and visualization, along with most of my data munging, happens in R. I recommend the following R install:

- [Microsoft R Open](https://mran.microsoft.com/open/). I was somewhat nervous when Microsoft took over the "Revolution R" distribution that became Microsoft R Open, but so far they have kept the software open-source. Compared to base R, there are two big perks: 1) automatic use of math kernel libraries to speed up performance on common tasks (make sure to install the MKL), and 2) checkpoints of the entire CRAN repository (which makes your code more reproducible by eliminating versioning problems).
- [RStudio](https://www.rstudio.com/products/rstudio/download/#download). My go-to IDE for most tasks.
- [Sublime Text 3](https://www.sublimetext.com) plus the [Sublime Studio](https://github.com/christophsax/SublimeStudio) plugin. My go-to IDE for heavy scripting tasks. Point SublimeStudio at your Microsoft R Open install, and when you code in sublimetext (a fantastic full-featured editor), you can highlight code and hit ctrl-enter and it will send the commands to an R console window (or open a plot etc). Supports markdown, help commands, and Shiny. Uses the same R install as RStudio (so there will be no weird package incompatibilities etc).

### R packages

