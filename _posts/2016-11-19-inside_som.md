---
layout: post
title:  "Inside a Self Organising Map"
date:   2016-11-19
comments: false
categories: som
---
  


When I first learned about Self Organising Maps I thought...cool name. Thereafter it became a way to try to understand data sets. Although I knew how, I never felt I quite understood why it works. So I have always been keen to look under the hood and see what the 'organisation' process looks like. So I forked [the Kohonen Package on the CRAN github mirror](https://github.com/cran/kohonen) and gave it a go.

# The SOM algorithm
There is a good description of the Self Organising Map (hereafter SOM) algorithm in [Elements of Statistical Learning](http://statweb.stanford.edu/~tibs/ElemStatLearn/) but I will give a less formal description. Moreover the kohonen package has lots of parameters and functionality which I will avoid to keep things simple. I just want to give a feel for how it all works.

Say we have a bunch of unlabeled data with each observation consisting of n numeric variables. Each observation can be thought of as a point in $$\mathbb{R}^{n}$$. The [iris](https://en.wikipedia.org/wiki/Iris_flower_data_set) data set is a good example (provided you remove the species column to make the data unlabeled...).


{% highlight r %}
dt.iris.unlabeled <- as.data.table(iris[, -which(names(iris) %in% c('Species'))])
dt.iris.unlabeled
{% endhighlight %}



{% highlight text %}
##      Sepal.Length Sepal.Width Petal.Length Petal.Width
##   1:          5.1         3.5          1.4         0.2
##   2:          4.9         3.0          1.4         0.2
##   3:          4.7         3.2          1.3         0.2
##   4:          4.6         3.1          1.5         0.2
##   5:          5.0         3.6          1.4         0.2
##  ---                                                  
## 146:          6.7         3.0          5.2         2.3
## 147:          6.3         2.5          5.0         1.9
## 148:          6.5         3.0          5.2         2.0
## 149:          6.2         3.4          5.4         2.3
## 150:          5.9         3.0          5.1         1.8
{% endhighlight %}

So in this case we can think of each row as a vector in $$\mathbb{R}^{4}$$. The goal of SOM is to find `k` points in $$\mathbb{R}^{n}$$ that each act as some sort of representative of a different subset of the data. This is the aim of many clustering algorithms. SOM goes further by putting the representatives together in a 2 dimensional grid. Specifically the representatives are enumerated and arranged in a grid. For example if k is 16 we can have a 4x4 grid.


{% highlight r %}
  plot(somgrid(xdim=4, ydim=4))
{% endhighlight %}

![plot of chunk plot-som-grid1](assets/img/inside_som/figure/plot-som-grid1-1.png)

Once the representatives are arranged like this they all have neighbours. Each representative has 8 neighbours unless it it on the edge in which case it has 3 (you can also use a hexagonal grid where each representative has 6 neighbours). The SOM algorithm works so that neighbours on the SOM grid stay close to each other in $$\mathbb{R}^{n}$$. The representatives are determined in such a way that if the they are close to each other in the map, then they will be near to each other in $$\mathbb{R}^{n}$$. So the grid is kind of like a 'map' of the feature space. Elements of Statistical Learning describes this in detail and in particular states that SOM can be thought of as a constrained form of K-means.

Anyway, let's look get back to the `iris` data and briefly show how SOM can be useful.

# Iris

First let's generate a map. I must stress the point here is not to show how to best produce a map, but rather to show how an SOM can be useful. I have just used the default configuration. Also in the kohonen algorithm these are called 'codes'.


{% highlight r %}
som.grid <- somgrid(5, 5)
# center and scale the data since we want to treat all features as equally important
dt.iris.unlabeled.scaled <- scale(dt.iris.unlabeled)
som.model <- som(as.matrix(dt.iris.unlabeled), 
                 som.grid, 
                 n.hood='square')
{% endhighlight %}

There are many different plots of the SOM but here is just one that I think shows how descriptive this technique can be. It uses the `plot.kohonen` `mapping` option to show the different species that have fallen into each SOM representative.


{% highlight r %}
# Here's a mapping of species to the id used to represent them on the map
print(data.table(Species=levels(iris$Species)))
{% endhighlight %}



{% highlight text %}
##       Species
## 1:     setosa
## 2: versicolor
## 3:  virginica
{% endhighlight %}



{% highlight r %}
plot(som.model, type="mapping", labels=as.numeric(iris$Species),
     col=as.numeric(iris$Species))
{% endhighlight %}

![plot of chunk iris-example-plot1](assets/img/inside_som/figure/iris-example-plot1-1.png)

Note that there is one code that doesn't have any flowers in it which isn't ideal - however this isn't really a problem for the current demonstration. From the plot we can make the following observations:

* The Setosa species (1s) seems to be separate from the other two species. This could be investigated further using the other kohonen plots. For instance we can plot the total distance of each code from it's neighbours with


{% highlight r %}
plot(som.model, type="dist.neighbours")
{% endhighlight %}

![plot of chunk iris-example-plot2](assets/img/inside_som/figure/iris-example-plot2-1.png)

We can see a boundary of sorts around the 4 Setosa species codes indicating a gap between these codes and the others. The gap is populated by a handful of the versicolor species (2s).

* The Versicolor and Virginica species can nearly be separated. We can use the plot to identify potentially interesting flowers by looking at codes with a mix of the two species.


{% highlight r %}
# Code 1 is on the bottom left with the code numbers incrementing horizontally 
# up to code 5. Code 6 is above code 1 on the next row up.
iris[som.model$unit.classif==16, ]
{% endhighlight %}



{% highlight text %}
##     Sepal.Length Sepal.Width Petal.Length Petal.Width    Species
## 73           6.3         2.5          4.9         1.5 versicolor
## 84           6.0         2.7          5.1         1.6 versicolor
## 120          6.0         2.2          5.0         1.5  virginica
## 124          6.3         2.7          4.9         1.8  virginica
## 127          6.2         2.8          4.8         1.8  virginica
## 134          6.3         2.8          5.1         1.5  virginica
## 147          6.3         2.5          5.0         1.9  virginica
{% endhighlight %}

We can also use the `codes` option of `plot.kohonen` to see what each of the codes look like.


{% highlight r %}
plot(som.model, type="codes")
{% endhighlight %}

![plot of chunk iris-example-plot3](assets/img/inside_som/figure/iris-example-plot3-1.png)

From this plot we can observe:

* The setosa species appears to vary mostly by sepal width with its petal length and petal width being smaller than the other species. 

* The versicolor and virginica species (2s and 3s in the plot above) seem to vary mostly by there sepal length and sepal width.

# Inside an SOM

Hoepfully the examples above show how an SOM can be useful. Now let's look at how the map 'organises' itself. Since we can't visualise 4 dimensions canonically let's simplify things just use two columns of the iris data set. Since going from 2 dimensions to 2 dimensions isn't very worthwhile, instead we will go from 2 dimensions to 1 i.e. we specify an 5x1 SOM grid.


{% highlight r %}
dt.iris.unlabeled.scaled.sepal.only <- dt.iris.unlabeled.scaled[, c('Sepal.Length', 'Sepal.Width')]
som.grid <- somgrid(5, 1, topo = 'rectangular')
som.model <- som(as.matrix(dt.iris.unlabeled.scaled.sepal.only), 
                 som.grid)
plot(som.model, type="mapping", labels=as.numeric(iris$Species),
     col=as.numeric(iris$Species))
{% endhighlight %}

![plot of chunk 2d-iris-example](assets/img/inside_som/figure/2d-iris-example-1.png)

This can only be known as a catepillar map...let's look at how it's formed

# One observation at a time

# Varying the learning rate





