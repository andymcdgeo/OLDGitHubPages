---
title: "Displaying Lithology Data on a Well Log Plot Using Python"
date: "2021-01-27"
draft: false
categories: ["Python", "Well Log Data"]
tags: ["petrophysics", "python", "lithology"]
featured_image: "/images/featured/2021-01-27 - Enhancing Visualisation of Well Log Plots With Plot Fills.png"
---
<!-- 
## Enhancing Visualization of Well Logs With Plot Fills

### **Applying Color Infill to Well Log Data Using matplotlib and fill_betweenx()**

![Log plot shading using fill_betweenx. [Image Created by Author]](https://cdn-images-1.medium.com/max/2800/1*Djx-t7XoP1fE8VtVIqOe1g.png) -->

Matplotlib is a great library to work with in Python and it is one that I always go back to time and time again to work with well logs. Due to its high degree of flexibility it can be tricky to get started with it at first, but once you have mastered the basics it can become a powerful tool for data visualization.

When working with well log data it can be common to apply color fills to the data to help quickly identify areas of interest. For example, identifying lithologies or hydrocarbon bearing intervals. Most of the time when I have been searching on the web for ways to achieve a color fill the articles point to filling between a line and the x-axis on a plot. There are significantly fewer results showing how to apply shading to well log plots, which generally have their longest axis along the y dimension, or to the y-axis in general.

This article forms part of my Python & Petrophysics series. Details of which can be found [here](http://andymcdonald.scot/python-and-petrophysics).

In this article, I am going to work through four different examples of how we can enhance the look of well log data using simple fills. These include:

* A simple color fill from a curve to the edge of a plot / track

* A color fill from a curve to both edges of a plot / track

* A variable fill from a curve to the edge of a plot / track

* A fill between two curves (density and neutron porosity) that changes when they cross over

For the examples below you can find my Jupyter Notebook and dataset on my GitHub repository at the following link.
[**andymcdgeo/Petrophysics-Python-Series**
*This series of Jupyter Notebooks take you through various aspects of working with Python and Petrophysical data. A…*github.com](https://github.com/andymcdgeo/Petrophysics-Python-Series)

## Applying Fills Using matplotlib

### Setting up The Libraries and Loading Data

To begin, we will import a number of common libraries before we start working with the actual data. For this article we will be using [pandas](https://pandas.pydata.org/), [matplotlib](https://matplotlib.org/) and [numpy](https://numpy.org/doc/stable/index.html). These three libraries allow us to load, work with and visualise our data. Additionally, the data that is commonly used to store and transfer data is .las files. For this we will use the excellent [lasio](https://github.com/kinverarity1/lasio) library to load this data. You can find more about this in my previous post.

 <iframe src="https://medium.com/media/3584d2c12e30e08e79d5f65f52aa248d" frameborder=0></iframe>

### Importing & Viewing LAS Data

The dataset we are using comes from the publicly available [Equinor Volve Field dataset](https://www.equinor.com/en/what-we-do/norwegian-continental-shelf-platforms/volve.html) released in 2018. The file used in this tutorial is from well 15/9- 19A which contains a nice set of well log data.

To begin loading in our las file, we can use the following method from lasio:

 <iframe src="https://medium.com/media/cfe6d844cac3f382e3ab36730eb8fd67" frameborder=0></iframe>

To make plotting easier, we will also convert our las file to a pandas dataframe and create a new column containing our depth curve, which is based on the dataframe index.

We can then find out what is contained within our dataset by calling upon the .describe() method for the dataframe like so.

 <iframe src="https://medium.com/media/b5e1b8c28fc7cd8a1a2ee7bb24ce8081" frameborder=0></iframe>

This will return back a simple, but very useful summary table detailing the statistics of all the curves.

![](https://cdn-images-1.medium.com/max/2250/1*aJOEmfOY_ZQdpM7PEL0dxA.png)

As we have added the depth curve as a column to the dataframe we can easily get the min and max values of our data. Note that this may not necessarily be the full extent of all the curves as seen in my previous post [Visualising Well Data Coverage Using Matplotlib.](https://towardsdatascience.com/visualising-well-data-coverage-using-matplotlib-f30591c89754)

### Plotting Our Data With A Simple Fill

Now that we have our data loaded and have confirmed we have the curves we want, we can begin plotting our data. For this article I am going to plot directly from the dataframe using the .plot() method.

In the code below, you will see that I am specifying a number of arguments:

* x & y axis

* c specifies the color of the line

* lw specifies the line width

* legend is used to turn on or off a legend. Useful to have on with multiple curves/lines

* figsize specifies the size of our figure in inches

 <iframe src="https://medium.com/media/06b334439a907034a271533a7871a931" frameborder=0></iframe>

The remaining sections of code allow the setting of the axes limits (ylim and xlim). Note that as we are plotting depth on our y-axis we have to flip the numbers around so that the deepest depth is the first number and the shallowest depth is the second number.

When we execute our code we get our plot:

![Gamma Ray data plotted using matplotlib.](https://cdn-images-1.medium.com/max/2000/1*2L4mMyMPs9aPxIt-kpwWmw.png)

We can enhance this plot a little bit further by adding a simple fill extending from the left edge of the plot to the curve value. This is achieved by using .fill_betweenx() .

To use this function we need to pass the y value (DEPTH), the curve being shaded to (GR) and the value we are shading from the GR curve to (0). We can then easily specify the color of the fill by using the facecolor argument.

 <iframe src="https://medium.com/media/6f91853bf193fc49663d104edbd936ad" frameborder=0></iframe>

When we run this code, we get a slightly nicer looking plot:

![Gamma Ray data with a color fill to the y-axis.](https://cdn-images-1.medium.com/max/2000/1*SKyoyL8gFwsoPamrq4iUGg.png)

We can go one step further and shade the opposite way by duplicating the line and changing the value to shade to along with the color:

 <iframe src="https://medium.com/media/63dd913e8b758f2e9514cdada45eae38" frameborder=0></iframe>

![Gamma ray plot shaded green to the left and yellow to the right allowing easy identification of clean and shaley intervals.](https://cdn-images-1.medium.com/max/2000/1*dY_JauJUVQ2R48ckU-aQBA.png)

Right away our plot is instantly better. We can quickly see where we have shalier intervals and where we have cleaner ones.

### Plotting Our Data With A Variable Fill

We can take our plot to the next level by applying a variable fill between the gamma ray curve and the y-axis. You will notice that the code below has expanded significantly compared to the code above.

We first have to identify how many colors we will split our shading into. This is done by assigning our x-axis values to a variable and working out the absolute difference between them using span = abs(left_col_value — right_col_value). This gives us our range of values.

We then grab a color map of our choosing using from a wide list of colormaps using cmap= plt.get_cmap('nipy_spectral'). A full list of colormaps can be found [here](https://matplotlib.org/3.1.0/tutorials/colors/colormaps.html). For this example, I have selected nipy_spectral.

The next section of code looks similar to the above with the exception that the x limits are now controlled by the variables left_col_value. and right_col_value. This allows us to change the value for the limits in just one place rather than in multiple places.

The final section, the for loop, loops through each of the color index values in the array that was created on line 14 and obtains a color from the color map. We then (line 26) use the fill_betweenx method to apply that color. Notice that we now using where = curve >= index in the arguments. This allows us to shade the appropriate color when the curve value is greater than or equal to the index value.

 <iframe src="https://medium.com/media/e6dd7c257eb6f5ebd11b3ddc846b53e6" frameborder=0></iframe>

Once we run the code we generate the following plot:

![Gamma Ray plot with a variable gradient fill using fill_betweenx.](https://cdn-images-1.medium.com/max/2000/1*RINrHUKZ9YWm8J2QWGIkQg.png)

At first glance, our plot is much better and allows to easily identify areas of similar values based on the color displayed.

### Applying Shading Between Two Curves With Different Scales

Our final example illustrates how we can apply lithology shading to our density and neutron porosity curves. These two curves are often shaded depending on the crossover. When density moves to the left of neutron porosity, we could potentially have a a porous reservoir rock. When the crossover occurs the opposite way, with density to the right of neutron porosity we could potentially have shale rock.

Note that this is very simplified and there are a number of different reasons why these two curves crossover one another.

The first step to display our density and neutron porosity data, is to add them to a plot. In this instance we have to create a figure and add multiple axes to it as opposed to using df.plot(). To allow us to plot the neutron porosity we have to add it on as an extra axis. Using ax1.twiny() we can share the depth curve between the two curves.

 <iframe src="https://medium.com/media/55905a0e6d22f9fbdd6462ca439831bd" frameborder=0></iframe>

![Density and neutron porosity on a matplotlib plot.](https://cdn-images-1.medium.com/max/2000/1*eRfUQtSlfVRUT3BUjYhSag.png)

We can now add the shading. Which is a little bit more complicated than expected as the two curves have different scales. Generally neutron porosity is scaled from 45 to -15 porosity units (p.u) (0.45 to -0.15 for decimal) and density from 1.95 to 2.95 g/cc.

We have to add in an extra bit of code that will scale one of the curves to the unit scale of the other. The section of code between lines 21 and 30 were obtained from the following stackoverflow posts: [here ](https://stackoverflow.com/questions/60658845/how-to-fill-areas-between-curves-with-different-scales-in-a-plot)and [here](https://stackoverflow.com/questions/65905036/filling-the-area-between-two-lines-on-different-scales-subplot-axes/65905776#65905776).

 <iframe src="https://medium.com/media/04b109acf94da7a76353039026ffc087" frameborder=0></iframe>

When we run this code we get the following plot.

![Variable color fill for neutron porosity and density data which are measured on different scale.](https://cdn-images-1.medium.com/max/2000/1*eMQGFEJ1t3p5hXl4q-E6AA.png)

This allows us to easily identify where potential reservoir sections are. To confirm what these sections are, further analysis would need to be carried out. You should always look at the other logging curves when doing an interpretation to aid your understanding and interpretation.

## Summary

In summary, matplotlib is a powerful data visualization tool when working with well log data. We can easily display our logs on tracks and fill in between the lines to aid visualization and interpretation of our data. In this article I have covered how to apply a fixed color fill and a variable gradient fill between our curve and the edge of the plot and also how to fill between two curves that are on different scales.

*“All images created by the author”*

***Thanks for reading!***

*If you have found this article useful, feel free to check out my other articles looking at various aspects of Python and well log data. You can also find my code used in this article and others at [**GitHub](https://github.com/andymcdgeo).***

*If you want to get in touch you can find me on [**LinkedIn](https://www.linkedin.com/in/andymcdonaldgeo/) **or at my [**website](http://andymcdonald.scot)**.*

*Interested in learning more about python and well log data or petrophysics? Follow me on [**Medium](https://andymcdonaldgeo.medium.com/)**.*
