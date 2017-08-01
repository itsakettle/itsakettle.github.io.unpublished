---
layout: post
title:  "Net and object distortion in Self Organising Maps"
date:   2017-08-01
comments: true
categories: som
feature: assets/img/som_distortion/som_distortion.png
---
  



Recently I read [this](https://link.springer.com/article/10.1007/BF00200832) paper about topology distortion in self organising maps by Li, Gasteiger and Zupan (1993). I thought it would be cool to try to illustrate the fundamental concepts of object and net distortion that are described in the paper using some simple examples.

Let's start by creating a square of exciting random data and fit a 1 dimensional self organising map with 15 codes (I'm using the `kohonen` package which calls a neuron of the SOM a code). If you're not familiar with self organising maps you might find the [previous post](https://www.itsakettle.com/inside_som) helpful.


{% highlight r %}
n <- 10000
dt <- data.table(x=runif(n), y=runif(n))
ggplot(dt) + 
  geom_point(aes(x=x, y=y), alpha=0.4) +
  ylim(0, 1) + 
  xlim(0, 1) +
  ggtitle('Square of random data')
{% endhighlight %}

![plot of chunk random_square](assets/img/som_distortion/figure/random_square-1.png)

{% highlight r %}
som.grid <- somgrid(1, 15, topo='rectangular')
som.model <- som(as.matrix(dt), grid=som.grid)
{% endhighlight %}

Here's a visualisation of the SOM showing a heatmap of the number of observations mapped to each code.


{% highlight r %}
plot(som.model, type='count')
{% endhighlight %}

![plot of chunk random_square_1d_map_plot](assets/img/som_distortion/figure/random_square_1d_map_plot-1.png)

The next visualisation shows which code each data point belongs to (a Voronoi diagram).


{% highlight r %}
dt.codes <-  as.data.table(som.model$codes)
dt.codes[, code := 1:.N]
dt[, code := som.model$unit.classif]

ggplot(dt, aes(x=x, y=y)) +
  geom_point(aes(color = as.factor(code)), size=0.5) +
  geom_label(data=dt.codes, 
             mapping=aes(x=x, y=y, label=code), size=6) +
  theme(legend.position="none") +
  ggtitle('Feature space')
{% endhighlight %}

![plot of chunk random_square_1d_feature_plot](assets/img/som_distortion/figure/random_square_1d_feature_plot-1.png)

# Object Distortion

The paper presents a continuous model of an SOM with the object and net distortion of a self organising map defined in terms of the discontinuity of functions. Specifically object distortion is the discontinuity of the function mapping the feature space to the SOM.

However in practice self organising maps are discrete and we can think of object distortion as when two codes that are near to each other in the feature space are far apart in the map. 

We can identify codes that are close to each other in the feature space but are not neighbours in the map using the Voronoi diagram above but this time we also show the arcs between neighbouring codes to help the eye. The function `SomGridArcs` that figures out the arcs can be found [here](https://github.com/itsakettle/blog-content/blob/master/som_distortion/lib/neighbours.R).


{% highlight r %}
dt.arcs <- SomGridArcs(som.model)
ggplot(dt, aes(x=x, y=y)) +
  geom_segment(data=dt.arcs, aes(x=start1, y=start2, xend=end1, yend=end2), alpha=.4) +
  geom_text(aes(label=code, color = as.factor(code)), size=2) +
  geom_label(data=dt.codes, 
             mapping=aes(x=x, y=y, label=code), 
             size=6) +
  theme(legend.position="none") +
  ggtitle('Feature space')
{% endhighlight %}

![plot of chunk random_square_1d_feature_plot_arcs](assets/img/som_distortion/figure/random_square_1d_feature_plot_arcs-1.png)

Observe that nodes 2 and 6 are beside each other in the feature space but there are 3 codes (3, 4 and 5) between them in the self organising map. This is an example of object distortion - there are many others like it!

# Net Distortion

Net distortion is the discontinuity of the function mapping the SOM to the feature space. We can think of net distortion as occuring when two neighbouring codes in the SOM map to points that are far apart in the feature space. 

Below is a slightly altered version of the U-matrix plot from the kohonen package where the distance between codes is indicated by the thickness of the line connecting the codes (I made these changes to a forked version of the kohonen package which can be found [here](https://github.com/itsakettle/kohonen)). The thicker the line, the closer the codes.


{% highlight r %}
plot(som.model, type='dist.neighbours', draw.arcs=TRUE)
{% endhighlight %}

![plot of chunk random_square_1d_map_umatrix](assets/img/som_distortion/figure/random_square_1d_map_umatrix-1.png)

Observe codes 14 and 15 are neighbours in the map but are far from each other in the feature space. The same somewhat applies for codes 5 and 6. Since we are in low dimensions it is possible to visualise the feature space.


{% highlight r %}
dt.arcs <- SomGridArcs(som.model)
ggplot(dt, aes(x=x, y=y)) +
  geom_segment(data=dt.arcs, aes(x=start1, y=start2, xend=end1, yend=end2), alpha=.4) +
  geom_text(aes(label=code, color = as.factor(code)), size=2) +
  geom_label(data=dt.codes, 
             mapping=aes(x=x, y=y, label=code), 
             size=6) +
  theme(legend.position="none") +
  ggtitle('Feature space')
{% endhighlight %}

![plot of chunk random_square_1d_feature_plot_arcs_2](assets/img/som_distortion/figure/random_square_1d_feature_plot_arcs_2-1.png)

We can see that when compared to the distance between other neighbouring codes there is a larger distance between codes 14 and 15 - points that are mapped to code 13 are even in the way.  However it doesn't exactly seem clear cut and for me it emphasises why net distortion is most easily described using a continuous SOM model where discontinuity is well defined. Let's try to demonstrate a clearer example.

# The letter T

In the paper Li, Gasteiger and Zupan use the letter T to illustrate object and net distortion. Let's start by making a T, plotting it and training an SOM.


{% highlight r %}
n <- 10000
t.thickness <- 0.1
dt <- data.table(x=runif(n), y=runif(n))
dt <- dt[!(x<0.5-(t.thickness/2) & y < (1 - t.thickness)),]
dt <- dt[!(x>0.5+(t.thickness/2) & y < (1 - t.thickness)),]


ggplot(dt) + 
  geom_point(aes(x=x, y=y), alpha=0.4) +
  ylim(0, 1) + 
  xlim(0, 1) +
  ggtitle('The letter T')
{% endhighlight %}

![plot of chunk the_letter_t](assets/img/som_distortion/figure/the_letter_t-1.png)


{% highlight r %}
som.grid <- somgrid(1, 15, topo='rectangular')
som.model <- som(as.matrix(dt), grid=som.grid)
dt.codes <-  as.data.table(som.model$codes)
dt.codes[, code := 1:.N]
dt[, code := som.model$unit.classif]

plot(som.model, type='dist.neighbours', draw.arcs=TRUE)
{% endhighlight %}

![plot of chunk random_t_1d_feature_plot_arcs](assets/img/som_distortion/figure/random_t_1d_feature_plot_arcs-1.png)

{% highlight r %}
dt.arcs <- SomGridArcs(som.model)
ggplot(dt, aes(x=x, y=y)) +
  geom_segment(data=dt.arcs, aes(x=start1, y=start2, xend=end1, yend=end2), alpha=.4) +
  geom_text(aes(label=code, color = as.factor(code)), size=2) +
  geom_label(data=dt.codes, 
             mapping=aes(x=x, y=y, label=code), 
             size=6) +
  theme(legend.position="none") +
  ggtitle('Feature space')
{% endhighlight %}

![plot of chunk random_t_1d_feature_plot_arcs](assets/img/som_distortion/figure/random_t_1d_feature_plot_arcs-2.png)

Again there are several examples of object distortion:

* Codes 9 and 4 are far from each other in the SOM but are close in the feature space (the same applies to several other codes and code 9).
* Codes 6 and 8 are beside each other in the feature space, but are not beside each other in the SOM.

There are two examples of net distortion here:

* Codes 6 and 7 are beside each other in the SOM but are separated by code 8 in the feature space. 
* There's a big leap between codes 8 and 9.

# So what?

An understanding of the object and net distortion of a self organising map might provide clues as to the structure of the higher dimensional data set that it represents. Object distortion seems to be related to the amount the 2 dimensional membrane of the map in the feature space has twisted back on itself. Net distortion is related to the fractures in this membrane.

In the examples above the mapping is easily visualised as the feature space was 2 dimensional - I'm interested in learning about measures and visualisations of object and net distortion when the feature space is of higher dimension. 



