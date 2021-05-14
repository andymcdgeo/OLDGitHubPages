---
title: "Displaying Lithology Data on a Well Log Plot Using Python"
date: "2021-02-07"
draft: false
categories: ["Python", "Well Log Data"]
tags: ["petrophysics", "python", "lithology"]
featured_image: "/images/featured/2021-01-27 - Lithology on Log Plots Using Python.png"
---

<!-- ### Using fill_betweenx() in matplotlib to add variable color fills and hatches for geological lithology data -->
<!-- 
![Well log plot with gamma ray, neutron porosity and bulk density data plotted alongside lithology data. Image created by the author.](https://cdn-images-1.medium.com/max/3000/1*OUPKh6RRocU7OL5cdfgWGQ.png) -->

Adding lithology information to a well log plot can enhance a petrophysical or geological interpretation. It can be used to understand why some log responses may behave the way they do. This data may have been sourced from a previous mineralogical interpretation or from mud logs.

In my previous article: [Enhancing Visualization of Well Logs With Plot Fills](https://towardsdatascience.com/enhancing-visualization-of-well-logs-with-plot-fills-72d9dcd10c1b), we saw: how to apply fixed color fill between a curve and the edge of a track, how to apply a density-neutron crossover fill and how to apply a variable fill based upon the value of curve being plotted.

In this article will cover how to use a variable fill to apply not only a color to our plot, but also a hatching. Using these two options allows us to generate lithological fill.

This article forms part of my Python & Petrophysics series. The full series can be found [here](http://andymcdonald.scot/python-and-petrophysics). For the examples below you can find my Jupyter Notebook and dataset on my GitHub repository at the following link.
[**andymcdgeo/Petrophysics-Python-Series**
*This series of Jupyter Notebooks take you through various aspects of working with Python and Petrophysical data. A…*github.com](https://github.com/andymcdgeo/Petrophysics-Python-Series)

To follow along, the Jupyter Notebook can be found at the link above and the data file for this article can be found in the [Data subfolder](https://github.com/andymcdgeo/Petrophysics-Python-Series/tree/master/Data) of the Python & Petrophysics repository.

## Setting Up & Displaying Lithology Fills on a Well Log Plot

### Setting up the Libraries

The first step is to bring in the libraries we will be working with for this article. We will be using just two libraries: [pandas](https://pandas.pydata.org/) and [matplotlib](https://matplotlib.org/). This will allow us to load our data to a dataframe and display it on a plot.

 <iframe src="https://medium.com/media/1ab7c14ac07111f53f84d34c844191b1" frameborder=0></iframe>

### Loading the Data

The dataset we will be using for this article comes from a recent Machine Learning competition for lithology prediction that was run by Xeek and FORCE ([https://xeek.ai/challenges/force-well-logs/overview](https://xeek.ai/challenges/force-well-logs/overview)). The objective of the competition was to predict lithology from a dataset consisting 98 training wells each with varying degrees of log completeness. The objective was to predict lithofacies based on the log measurements. To download the file, navigate to the Data section of the link above.

The data can be loaded using pd.read_csv(). As this is quite a large dataset, we will use a single well from it to work with and we will also take just the curves (columns) we need. We can do that by taking a subset of the data like so:

 <iframe src="https://medium.com/media/9ed60c41dddf48e1f1c429ecaea4097a" frameborder=0></iframe>

To make things simpler we will rename the FORCE_2020_LITHOFACIES_LITHOLOGY column to just LITHOLOGY . This is done by using the rename function on the dataframe and passing a dictionary of the old name and the new name. Note that using the inplace=True argument will replace the column name in the original dataframe.

 <iframe src="https://medium.com/media/a229d8e2239af4d423ee85a35c76651f" frameborder=0></iframe>

When we call upon our dataframe using data.head() we can see that our dataset is small and our lithology column has been renamed.

![The first five rows of our pandas dataframe containing Gamma Ray (GR), Bulk Density (RHOB), Neutron Porosity (NPHI) and Lithology.](https://cdn-images-1.medium.com/max/2000/1*V7fLIOX2b14Wzqmi9l-IuQ.png)

### Setting up the Lithologies Using a Nested Dictionary

The lithologies currently listed in the lithology column contain a series of numbers. We can map these in a nested dictionary where we have the full name of the lithology, a simplified number (if required for conversion), the hatch style, and the color of the fill.

The colors are based on the Kansas Geological Survey [website](http://www.kgs.ku.edu/PRS/Ozark/PROFILE/HELP/DATA_ENTRY/lithology/Lithology-Symbols.html). However, hatching symbols are limited in the default setup of matplotlib. In a future article, I will cover how to create custom hatches and lithologies.

 <iframe src="https://medium.com/media/66f09450905b501e34bec2ae8d35c39d" frameborder=0></iframe>

We can quickly convert our nested dictionary to a pandas dataframe to make it easier for humans to read by using the from_dict function like so.

 <iframe src="https://medium.com/media/78e0788dc25f2764e00cb55da9eeffba" frameborder=0></iframe>

This will return a nicely formatted table where we can see the colors and hatches for each lithology code.

![A pandas dataframe generated from a nested dictionary.](https://cdn-images-1.medium.com/max/2000/1*8c2u7UC9ivR1pmI1_An77w.png)

By just looking at the dictionary, it can be difficult to understand how each of these will look. To solve this, we can quickly create a simple image showing the different hatchings and colors for each lithology.

To do this, we will first define some co-ordinates for two new variables x and y. Then we will setup the matplotlib figure and axes using subplots:

fig, axes = plt.subplots(ncols=4, nrows=3, sharex=True, sharey=True, fig size=(10,5), subplot_kw={'xticks':[], 'yticks':[]})

This allows us to define the size of the plot (4 columns by 3 rows), the figure size, and turning off the tick marks on both axes.

We then want to loop through the flattened axes and the nested dictionary to add the data and hatched color fills to each subplot. As we are dealing with a nested dictionary, we need to us square brackets multiple times [ ][ ] to go in two levels. The first level is the key, and the second level is the key of the nested dictionary. For example, to retrieve the color we can access it by typing in lithology_numbers[30000]['color']

 <iframe src="https://medium.com/media/fa1229dbe36efdffe309ed2a6f5a5466" frameborder=0></iframe>

This generates the following legend that we can refer to later on. Notice that the lithology symbols do not 100% match the Kansas Geological Survey chart. We will see how to resolve this in a future article.

![Graphical representation using matplotlib of our colors and hatches. Image created by author.](https://cdn-images-1.medium.com/max/3000/1*0tZaVbCGjs1Umeqd0EoUKg.png)

### Setting up the Well Log Plot With a Lithology Track

In previous articles ([Enhancing Visualization of Well Logs With Plot Fills](https://towardsdatascience.com/enhancing-visualization-of-well-logs-with-plot-fills-72d9dcd10c1b) and [Exploratory Data Analysis With Well Log Data](https://towardsdatascience.com/exploratory-data-analysis-with-well-log-data-98ad084c4e7)) I have set the plot up on the fly and passed in the dataframe I am working with. In this article, I will be creating a simple makeplot function which we can pass the dataframe into.

This allows us to easily reuse the code for other dataframes, assuming the curve names are the same. This could be further refined to make it more generic. This new function takes in three arguments, the dataframe, the top_depth, and the bottom depth.

This code will generate a well log plot with 3 tracks one for the Gamma Ray, one containing both Neutron Porosity and Bulk Density, and the final one containing our geological lithology data.

 <iframe src="https://medium.com/media/428eaaf16798593ae80c2d73f9b88246" frameborder=0></iframe>

The key piece of code to ensure our lithology fill_betweenx function from matplotlib creates a variable fill is shown below. All we are doing is looping over each of the keys within the dictionary, assigning simpler variable names to the features within the dictionary, and then using the fill_betweenx function to check if the value in our lithology column equals the dictionary key. If it does, then it will apply the relevant color and hatching to the fill. It will then move on to the next key and repeat until all keys have been cycled through.

 <iframe src="https://medium.com/media/963e25c6171eb2a1c9b005b0d19dc40d" frameborder=0></iframe>

### Displaying the Well Log Plot & Lithology

Once we have the plot set up as a function. We can simply pass in our dataframe and the depths that we want to plot. In this example, we will focus on a small section of the data so that we are able to see the change in the plot fills.

 <iframe src="https://medium.com/media/83e482152e2edd11717bc853a764aab6" frameborder=0></iframe>

Once we do this, the plot below will appear.

![Well log plot generated by matplotlib with geological lithology data covering 2600 to 3100m. Image created by the author.](https://cdn-images-1.medium.com/max/4500/1*-pxzLmkbuzwxnWXOp8ALRA.png)

To see the lithology fill in more detail, we can zoom in by changing the depth range values.

![Well log plot generated by matplotlib with geological lithology data covering 2900 to 3100m. Image created by the author.](https://cdn-images-1.medium.com/max/4500/1*i6Kdqh3tQHy-49-z56n6LQ.png)

## Summary

In this article we have covered how to setup and display lithology data using matplotlib’s fill_betweenx() function. We saw how a nested dictionary can be used to store parameters for each lithology and then be called upon using a for loop to fill with the correct color and hatch.

Adding this data to a plot can enhance how the geoscientist or petrophysicist relates the electrical logging measurements to the geology.

***Thanks for reading!***

*If you have found this article useful, please feel free to check out my other articles looking at various aspects of Python and well log data. You can also find my code used in this article and others at [**GitHub](https://github.com/andymcdgeo).***

*If you want to get in touch you can find me on [**LinkedIn](https://www.linkedin.com/in/andymcdonaldgeo/) **or at my [**website](http://andymcdonald.scot/)**.*

*Interested in learning more about python and well log data or petrophysics? Follow me on [**Medium](https://andymcdonaldgeo.medium.com/)**.*

## References

Kansas Geological Survey, Lithology Symbols. [http://www.kgs.ku.edu/PRS/Ozark/PROFILE/HELP/DATA_ENTRY/lithology/Lithology-Symbols.html](http://www.kgs.ku.edu/PRS/Ozark/PROFILE/HELP/DATA_ENTRY/lithology/Lithology-Symbols.html)

“Lithofacies data was provided by the FORCE Machine Learning competition with well logs and seismic 2020” Bormann P., Aursand P., Dilib F., Dischington P., Manral S. 2020. 2020 FORCE Machine Learning Contest. [https://github.com/bolgebrygg/Force-2020-Machine-Learning-competition](https://github.com/bolgebrygg/Force-2020-Machine-Learning-competition)
