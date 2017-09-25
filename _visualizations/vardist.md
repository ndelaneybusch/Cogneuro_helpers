---
title: Visualizing the distribution of continuous variables
---
# Description
Makes a [kernel density plot](http://www.mvstat.net/tduong/research/seminars/seminar-2001-05/) out of a given variable of a given data frame, along with a rug of all data points along the bottom (comment this line out if not needed). Puts a vertical line at the mean, and lists in the n along the bottom. Quick option for plotting the log-transformed variable instead.

# Dependencies
{% highlight r %}
library("ggplot2")
library("ggthemes")
{% endhighlight %}

# Function
{% highlight r %}
plot_var_dense <- function(dataframe_name, varname, apply_log = FALSE)
{
  dataframe_name$toplot <- dataframe_name[,varname]
  if (apply_log==TRUE) {dataframe_name$toplot <- log(dataframe_name$toplot)}
  ggplot(dataframe_name[complete.cases(dataframe_name$toplot),], aes(toplot))+
    geom_density(fill='blue',alpha = 0.2, adjust=0.7)+
    geom_rug(col="coral3",alpha=0.6)+
    geom_vline(col="coral3",alpha=.8,size=2,xintercept=mean(dataframe_name$toplot, na.rm = TRUE))+
    theme_fivethirtyeight()+ylab("Relative Density")+xlab(paste0(varname, ", n=", sum(!is.na(dataframe_name$toplot))))+
    labs(title=paste0("Density + Rug Plot: ",varname))+theme(plot.title = element_text(size = rel(2)))+
    theme(axis.title=element_text(size="16"))+theme(axis.title.y = element_text(angle=90))
} 
{% endhighlight %}

# Function Call
{% highlight r %}
plot_var_dense(df_behavioral, "RT")
plot_var_dense(df_behavioral, "RT", apply_log = TRUE)
{% endhighlight %}

__Plot 1__
![vardist_1](/Cogneuro_helpers/img/vardist_1.png)  
  
  
__Plot 2 (log transformed)__
![vardist_2](/Cogneuro_helpers/img/vardist_2.png)
