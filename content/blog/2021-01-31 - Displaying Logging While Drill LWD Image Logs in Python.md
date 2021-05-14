---
title: "Displaying Lithology Data on a Well Log Plot Using Python"
date: "2021-01-31"
draft: false
categories: ["Python", "Well Log Data"]
tags: ["petrophysics", "python", "lithology"]
featured_image: "/images/featured/2021-01-31-Displaying Logging While Drilling LWD Images in Python.png"
---

<!-- ![Logging While Drilling image data displayed using matplotlib in Python. Image created by the author.](https://cdn-images-1.medium.com/max/3600/1*CItTvpKJk860vwc6HMLRkQ.png) -->

## Introduction

Borehole image logs are false-color pseudo images of the borehole wall generated from different logging measurements/tools. How borehole images are acquired differs between wireline logging and logging while drilling (LWD). In the wireline environment measurements are made from buttons on pads that are pressed up against the borehole wall and provide limited coverage, but at a high resolution. In contrast, in the LWD environment measurements are made from sensors built into tools that form part of the drillstring/tool assembly, and using the tool rotation, provide full 360-degree coverage. LWD image data is often split into sectors, the number of which will vary depending on the tool technology. As the tool rotates the data is binned into the relevant sector and from it we can build up a pseudo image of the borehole wall.

The generated images are often viewed in two dimensions on a log plot as an ‘unwrapped borehole’ and as seen in the image above. The cylindrical borehole is cut along the north azimuth in vertical wells or along the highside of the borehole in deviated/horizontal wells. As a result of being projected onto a 2D surface, any planar features that intersect the borehole are represented as sinusoid shapes on the plot. By analyzing the amplitude and offset of these sinusoids geologists can gain an understanding of the geological structure of the subsurface. Borehole image data can also be used to identify and classify different geological facies/textures, identify thin-beds, fault and fracture analysis, and more.

In this article, I am going to work through displaying logging while drilling image data from azimuthal gamma ray and azimuthal density measurements using Python and matplotlib.

This article forms part of my Python & Petrophysics series. Details of which can be found [here](http://andymcdonald.scot/python-and-petrophysics). For the examples below you can find my Jupyter Notebook and dataset on my GitHub repository at the following link.
[**andymcdgeo/Petrophysics-Python-Series**
*This series of Jupyter Notebooks take you through various aspects of working with Python and Petrophysical data. A…*github.com](https://github.com/andymcdgeo/Petrophysics-Python-Series)

To follow along, the data file for this article can be found in the [Data subfolder](https://github.com/andymcdgeo/Petrophysics-Python-Series/tree/master/Data) of the Python & Petrophysics repository.

## Loading and Displaying LWD Image Data

### Setting up The Libraries and Loading Data

Before we begin working with our data we will need to import a number of libraries for us to work with. For this article, we will be using [pandas](https://pandas.pydata.org/), [matplotlib](https://matplotlib.org/), [numpy](https://numpy.org/doc/stable/index.html) and [lasio](https://github.com/kinverarity1/lasio) libraries. These three libraries allow us to load our las data, work with it and create visualizations of the data.

 <iframe src="https://medium.com/media/ae50d4cd82575dbbc86b2c8964c69f76" frameborder=0></iframe>

### Importing Well and Survey Data

The dataset we are using comes from the publicly accessible Netherlands [NLOG databas](https://www.nlog.nl/en/welcome-nlog)e. The las file contains a mixture of LWD measurements and two images. As las files are flat and don’t support arrays, LWD image data is often delivered as individual sectors. The second file we will load is the survey, this allows us to understand the deviation and azimuth of the wellbore and is useful for calculating accurate formation dips. We will not be covering dip picking in this article.

We will first load the las file using lasio and convert it to a dataframe using .df().

 <iframe src="https://medium.com/media/dbea8a08295388db8bcb3fb736650ada" frameborder=0></iframe>

Once the data has been loaded, we can confirm the data that we have by using .describe(). As there is a large number of curves present within the file, we can call upon df.columns() to view the full list of curves present:

 <iframe src="https://medium.com/media/eda25a684d6706a239a4b0aa0a097d5a" frameborder=0></iframe>

Which returns:

    Index(['APRESM', 'GRAFM', 'RACELM', 'RPCELM', 'RACEHM', 'RPCEHM', 'RACESLM', 'RPCESLM', 'RACESHM', 'RPCESHM', 'RPTHM', 'NPCKLFM', 'DPEFM', 'BDCFM', 'DRHFM', 'TVD', 'BLOCKCOMP', 'INNM', 'ROP_AVG', 'WOB_AVG', 'TCDM', 'ABDCUM', 'ABDCLM', 'ABDCRM', 'ABDCDM', 'ABDC1M', 'ABDC2M', 'ABDC3M', 'ABDC4M', 'ABDC5M', 'ABDC6M', 'ABDC7M', 'ABDC8M', 'ABDC9M', 'ABDC10M', 'ABDC11M', 'ABDC12M', 'ABDC13M', 'ABDC14M', 'ABDC15M', 'ABDC16M','ABDCM', 'GRAS0M', 'GRAS1M', 'GRAS2M', 'GRAS3M', 'GRAS4M', 'GRAS5M', 'GRAS6M', 'GRAS7M', 'GRASM'], dtype='object')

As explained in the introduction, we can see the azimuthal density image is split into 16 individual sectors labeled ABDC1M to ABDC16M. The azimuthal gamma ray image is split into 8 sectors and labeled GRAS0M to GRAS7M.

The survey data is contained within a csv file and can be loaded as follows.

 <iframe src="https://medium.com/media/42daae0f32d51b13e63b356b57904686" frameborder=0></iframe>

When we read the columns from the dataframe we find we have three curves present: measured depth (DEPTH), hole deviation (DEVI), and hole azimuth (AZIM). We will be using these curves for additional visualization on our plot.

Before we plot the data, we will make the data simpler to handle by splitting it the main dataframe into two smaller ones for each image. We can achieve this by calling upon the dataframe (df) and supplying a list of curve names that we want to extract.

 <iframe src="https://medium.com/media/4a49f8d280bb735cdaa5f2b0416fd6a7" frameborder=0></iframe>

## Plotting the Image Data

### Azimuthal Density Image

Plotting the image data is fairly straight forward. We first have to create a figure object using plt.figure(figsize=(7,15)). The figsize argument allows us to control the size, in inches, of our final plot.

We can also, optionally, create two new variables for our min and max depth. These are used to define the extent of the plotting area. As the index of our dataframe contains our depth values we can create two new variables miny and maxy that are equal to the minimum and maximum index values.

 <iframe src="https://medium.com/media/3a6a09a4e296a0bfe0be2ebf65bf48a2" frameborder=0></iframe>

When we execute the code we generate an image that shows us our features, but at the same time, we can see it is a bit blocky looking. If you look close enough you will be able to make out the individual sectors for the image.

![Azimuthal density image plotted using matplotlib imshow() with no interpolation.](https://cdn-images-1.medium.com/max/2000/1*dIHzBydtk0ONVHFh9spRvg.png)

We can apply some interpolation to the image and smooth it out. In the example below, the interpolation has been changed to bilinear.

![Azimuthal density image plotted using matplotlib imshow() with bilinear interpolation.](https://cdn-images-1.medium.com/max/2000/1*pf8igbPD1MbvaN8Yh8WUJg.png)

You can find a full list of options for interpolation available in the [matplotlib documentation](https://matplotlib.org/3.1.1/gallery/images_contours_and_fields/interpolation_methods.html). To illustrate the different methods, I have used the code from the documentation to generate a grid show the different options.

 <iframe src="https://medium.com/media/9c6a45aac0a3975b156ddc3ec7bd4245" frameborder=0></iframe>

![Different types of interpolation of imshow() and azimuthal density image data.](https://cdn-images-1.medium.com/max/3300/1*y8PTgjmZndYV-fwzSCTNOw.png)

You can see from the image above that some (lanczos and sinc) appear slightly sharper than the others. The choice will ultimately be down to user preference when working with the image data.

### Azimuthal Gamma Ray Image

We can repeat the above code, by change the data frame from azidendf to azigamdf to generate our Azimuthal Gamma Ray image.

 <iframe src="https://medium.com/media/2ee7d0183b95644e283a71f635e5adcb" frameborder=0></iframe>

![Azimuthal gamma ray image plotted using matplotlib imshow().](https://cdn-images-1.medium.com/max/2000/1*8FZ2-qMYcPd7ubvQI2mEwA.png)

When we view our image, we notice that the level of detail is significantly less compared to the density image. This is related to the measurement type and the lower number of sectors being recorded.

## Building The Final Plot

We can now plot both our image logs together alongside the survey data using subplots. The subplot2grid((1,3),(0,0)) method allows us to set up the shape of the plot, in this case, we are plotting 1 row and 3 columns. We then assign ax1, ax2, and ax3 to the three columns as denoted by the second set of numbers in the function. As we are plotting deviation on the same track as the azimuth we need to use ax.twiny() to add a new axis on top of an existing one.

To plot our image data, we can use the code in the previous sections for each of the images and instead of assigning it to a plot, we can assign them to subplot axes.

 <iframe src="https://medium.com/media/d1a683fb0eb1a13f22fa072b98156b22" frameborder=0></iframe>

![Matplotlib plot of azimuthal density and azimuthal gamma ray data plotted alongside wellbore deviation and azimuth.](https://cdn-images-1.medium.com/max/3000/1*ryk3ByMiou4OtIjalZOoYA.png)

Now we can see both images together and we can also take into account our borehole deviation, which indicates that we are in a horizontal section. This is important to know, especially when calculating any dips from image logs.

You may notice that the bed at 2,500 appears to offset between the two plots and would warrant further investigation. This will not be covered in this article.

## Summary

In this article, we have covered how to load and display borehole image logs from LWD azimuthal gamma ray and density measurements. Once the data has been separated into their respective dataframes it is easy to pass them to the imshow() plot in matplotlib. From this image data geologists and petrophysicists can gain a better understanding of the geological makeup of the subsurface.

*“All images generated by the author”*

***Thanks for reading!***

*If you have found this article useful, please feel free to check out my other articles looking at various aspects of Python and well log data. You can also find my code used in this article and others at [**GitHub](https://github.com/andymcdgeo).***

*If you want to get in touch you can find me on [**LinkedIn](https://www.linkedin.com/in/andymcdonaldgeo/) **or at my [**website](http://andymcdonald.scot/)**.*

*Interested in learning more about python and well log data or petrophysics? Follow me on [**Medium](https://andymcdonaldgeo.medium.com/)**.*
