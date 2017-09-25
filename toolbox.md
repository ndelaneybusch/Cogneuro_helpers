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
Here is the code to obtain my recommended R packages:

{% highlight r %}
install.packages("tidyverse")
install.packages("ggthemes")
install.packages("shiny")
install.packages("doParallel")
install.packages("foreign")
install.packages("doBy")
install.packages("Hmisc")
install.packages("caret")
install.packages("car")
install.packages("lmerTest")
install.packages("itsadug")
install.packages("rgl")
install.packages("psych")
{% endhighlight %}

__Descriptions__:
- [tidyverse](https://www.tidyverse.org/packages/) is a large set of packages by Hadley Wickham and colleagues to make everyday data manipulation tasks easier. I do most of my work in the tidyverse. There are a few packages in particular that come with tidyverse that deserve special mention, and are well worth learning to use effectively:
  - [ggplot2]() is a language of visualization that you can use to build simple, beautiful graphics. I highly recommend [reading this chapter](http://r4ds.had.co.nz/data-visualisation.html). Good places to grab code include [this cookbook](http://www.cookbook-r.com/Graphs/) and [this tutorial](http://tutorials.iq.harvard.edu/R/Rgraphics/Rgraphics.html). [ggthemes](https://cran.r-project.org/web/packages/ggthemes/vignettes/ggthemes.html) is a lovely set of pre-packaged visual design property sets that you can apply to existing plots to change the way they look in coordinated ways.
  - [magrittr](http://magrittr.tidyverse.org) fundamentally changes the way that sequences of commands are applied to data. Rather than nesting functions inside of one another (making big incomprehensible lines of code), you sequence the commands from left to right (like a sentence) and pass the output of one function to the input of the next function using "pipes", denoted by %>%. See chapter [here](http://r4ds.had.co.nz/pipes.html).
  - [tidyr](http://tidyr.tidyverse.org) is an easy way to switch between wide and long data formats.
  - [dplyr](http://dplyr.tidyverse.org) great for selecting/filtering your data and computing new variables. See book chapter [here](http://r4ds.had.co.nz/transform.html#grouped-mutates-and-filters).
  - There are specialized helper functions for working with [strings](http://stringr.tidyverse.org) ([chapter](http://r4ds.had.co.nz/strings.html)), [dates](http://lubridate.tidyverse.org) ([chapter](http://r4ds.had.co.nz/dates-and-times.html)), and [factors](http://forcats.tidyverse.org) ([chapter](http://r4ds.had.co.nz/factors.html)).
- [Shiny](https://shiny.rstudio.com) apps are a great way to explore your data with an interactive GUI. Building them is fairly simple once you get the hang of it.
- [doParallel](https://cran.r-project.org/web/packages/doParallel/vignettes/gettingstartedParallel.pdf) is an incredibly simple way to use more than 1 CPU whenever you run a "for" loop.
- Learn how to use the Foreign package to [import SAS/SPSS/Stata/Minitab/Octave files into R](http://www.statmethods.net/input/importingdata.html). [Readr](http://readr.tidyverse.org) is a great go-to for less exotic file formats, specifically any comma/tab/space delimited file ([chapter](http://r4ds.had.co.nz/data-import.html)).
- [doBy](https://cran.r-project.org/web/packages/doBy/vignettes/doBy.pdf) is my preferred way to implement group-wise functions.
- [Hmisc](http://data.vanderbilt.edu/fh/R/Hmisc/examples.nb.html) is a handy set of helper functions that make a wide variety of tasks easier when exploring data ([reference card](http://biostat.mc.vanderbilt.edu/wiki/pub/Main/Hmisc/Hmisc-refcard.pdf)).
  
### RStudio Plugins

### R Reproducibility
- Use the [project](http://r4ds.had.co.nz/workflow-projects.html) functionality in RStudio to keep dependencies and data sources straight.
- Use the [Git](https://support.rstudio.com/hc/en-us/articles/200532077-Version-Control-with-Git-and-SVN) functionality in RStudio to make good version control habits easy.
