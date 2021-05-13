---
title: "Loading Multiple Well Log LAS Files Using Python"
date: "2021-02-14"
draft: false
categories: ["Petrophysics", "Python"]
tags: ["petrophysics", "python", "las files"]
featured_image: "/images/featured/2021-02-14_Loading-multiple-well-log-las-files-using-python.png"
---

### Appending Multiple LAS Files to a Pandas Dataframe

![Crossplots of density vs neutron porosity from multiple wells using the Python library matplotlib. Imagae created by the author.](https://cdn-images-1.medium.com/max/3600/1*1VrRe0q5TAIgz22EXlPehQ.png)

Log ASCII Standard (LAS) files are a common Oil & Gas industry format for storing and transferring well log data. The data contained within is used to analyze and understand the subsurface, as well as identify potential hydrocarbon reserves. In my previous article: [Loading and Displaying Well Log Data](https://andymcdonaldgeo.medium.com/loading-and-displaying-well-log-data-b9568efd1d8), I covered how to load a single LAS file using the LASIO library.

In this article, I expand upon that by showing how to load multiple las files from a subfolder into a single [pandas dataframe](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html). Doing this allows us to work with data from multiple wells and visualize the data quickly using matplotlib. It also allows us to prepare the data in a single format that is suitable for running through Machine Learning algorithms.

This article forms part of my Python & Petrophysics series. Details of the full series can be found [here](http://andymcdonald.scot/python-and-petrophysics). You can also find my Jupyter Notebooks and datasets on my GitHub repository at the following link.
[**andymcdgeo/Petrophysics-Python-Series**
*This series of Jupyter Notebooks take you through various aspects of working with Python and Petrophysical data. A…*github.com](https://github.com/andymcdgeo/Petrophysics-Python-Series)

To follow along with this article, the Jupyter Notebook can be found at the link above and the data file for this article can be found in the [Data subfolder](https://github.com/andymcdgeo/Petrophysics-Python-Series/tree/master/Data) of the Python & Petrophysics repository.

The data used for this article originates from the publicly accessible [Netherlands NLOG Dutch Oil and Gas Portal](https://www.nlog.nl/en/welcome-nlog).

## Setting up the Libraries

The first step is to bring in the libraries we will be working with. We will be using five libraries: [pandas](https://pandas.pydata.org/), [matplotlib](https://matplotlib.org/), [seaborn](https://seaborn.pydata.org), [os](https://docs.python.org/3/library/os.html), and [lasio](https://lasio.readthedocs.io).

Pandas, os and lasio will be used to load and store our data, whereas matplotlib and seaborn will allow us to visualize the contents of the wells.

 <iframe src="https://medium.com/media/c54608c7b1796d60651336fef828fa50" frameborder=0></iframe>

Next we are going setup an empty list which will hold all of our las file names.

Secondly, in this example we have our files stored within a sub folder called Data/15-LASFiles/. This will change depending on where your files are stored.

 <iframe src="https://medium.com/media/343b716c6bdea570fd789102445980fd" frameborder=0></iframe>

We can now use the os.listdir method and pass in the file path. When we run this code, we will be able to see a list of all the files in the data folder.

 <iframe src="https://medium.com/media/cff6e806d752a03b02fd068cc2f588e8" frameborder=0></iframe>

From this code, we get a list of the contents of the folder.

    ['L05B03_comp.las',
     'L0507_comp.las',
     'L0506_comp.las',
     'L0509_comp.las',
     'WLC_PETRO_COMPUTED_1_INF_1.ASC']

## Reading the LAS Files

As you can see above, we have returned 4 LAS files and 1 ASC file. As we are only interested in the LAS files we need to loop through each file and check if the extension is .las. Also, to catch any cases where the extension is capitalized (.LAS instead of .las), we need to call upon .lower() to convert the file extension string to lowercase characters. 
 
Once we have identified if the file ends with .las, we can then add the path (‘Data/15-LASFiles/’) to the file name. This is required for lasio to pick up the files correctly. If we only passed the file name, the reader would be looking in the same directory as the script or notebook, and would fail as a result.

 <iframe src="https://medium.com/media/8af2ece450f29d348b384c55c0ceb47c" frameborder=0></iframe>

When we call the las_file_list we can see the full path for each of the 4 LAS files.

    ['Data/15-LASFiles/L05B03_comp.las',
     'Data/15-LASFiles/L0507_comp.las',
     'Data/15-LASFiles/L0506_comp.las',
     'Data/15-LASFiles/L0509_comp.las']

## Appending Individual LAS Files to a Pandas Dataframe

There are a number of different ways to concatenate and / or append data to dataframes. In this article we will use a simple method of create a list of dataframes, which we will concatenate together.

First, we will create an empty list using df_list=[]. Then secondly, we will loop through the las_file_list, read the file and convert it to a dataframe.

It is useful for us to to know where the data originated. If we didn’t retain this information, we would end up with a dataframe full of data with no information about it’s origins. To do this, we can create a new column and assign the well name value: lasdf['WELL']=las.well.WELL.value. This will make it easy to work with the data later on.

Additionally, as lasio sets the dataframe index to the depth value from the file, we can create an additional column called DEPTH.

 <iframe src="https://medium.com/media/614b543f4884b9892b19f8f0837e94be" frameborder=0></iframe>

We will now create a working dataframe containing all of the data from the LAS files by concatenating the list objects.

 <iframe src="https://medium.com/media/a57b9965a38307fc6bc558f84c7dc42e" frameborder=0></iframe>

When we call upon the working dataframe, we can see that we have our data from multiple wells in the same dataframe.

![Pandas dataframe compiled from multiple las files.](https://cdn-images-1.medium.com/max/2000/1*Qtpmo7lLXdWUTwXdSbksZw.png)

We can also confirm that we have all the wells loaded by checking for the unique values within the well column:

 <iframe src="https://medium.com/media/38e8a44467956911784eb36840633b73" frameborder=0></iframe>

Which returns an array of the unique well names:

    
    array(['L05-B-03', 'L05-07', 'L05-06', 'L05-B-01'], dtype=object)

If our LAS files contain different curve mnemonics, which is often the case, new columns will be created for each new mnemonic that isn’t already in the dataframe.

## Creating Quick Data Visualizations

Now that we have our data loaded into a pandas dataframe object, we can create some simple and quick multi-plots to gain insight into our data. We will do this using crossplot/scatterplots, a boxplot and a Kernel Density Estimate (KDE) plot.

To start this, we first need to group our dataframe by the well name using the following:

 <iframe src="https://medium.com/media/827ca5585603dd42e0a031f670e15642" frameborder=0></iframe>

### Crossplot / Scatterplots Per Well

Crossplots (also known as scatterplots) are used to plot one variable against another. For this example we will use a neutron porosity vs bulk density crossplot, which is a very common plot used in petrophysics.

Using a similar piece of code that was previously mentioned on my [Exploratory Data Analysis With Well Log Data](https://towardsdatascience.com/exploratory-data-analysis-with-well-log-data-98ad084c4e7) article, we can loop through each of the groups in the dataframe and generate a crossplot (scatter plot) of neutron porosity (NPHI) vs bulk density (RHOB).

 <iframe src="https://medium.com/media/44e8b2362fbf0caada1da99a79cd3605" frameborder=0></iframe>

This generates the following image with 4 subplots:

![Crossplots of density vs neutron porosity from multiple wells using the Python library matplotlib. Image created by the author.](https://cdn-images-1.medium.com/max/3000/1*McO4fbnkOmf2ZWDR-Hk34Q.png)

### Boxplot of Gamma Ray Per Well

Next up, we will display a boxplot of the gamma ray cuvre from all wells. The box plot will show us the extent of the data (minimum to maximum), the quartiles, and the median value of the data.

This can be achieved using a single line of code in the seaborn library. In the arguments we can pass in the workingdf dataframe for data, and the WELL column for the hue. The latter of which will split the data up into individual boxes, each with it’s own unique color.

 <iframe src="https://medium.com/media/a13a12d08a128c69512a2a8a2a3a7acd" frameborder=0></iframe>

![Boxplot of Gamma Ray across 4 separate wells. Image created by the author.](https://cdn-images-1.medium.com/max/2000/1*dJAD9Sn7crtWk6IDnbnijg.png)

### Histogram (Kernel Density Estimate)

Finally, we can view the distribution of the values of a curve in the dataframe by using a Kernel Density Estimate plot, which is similar to a histogram.

Again, this example shows another way to apply the groupby function. We can tidy up the plot by calling up matplotlib functions to set the x and y limits.

 <iframe src="https://medium.com/media/5698e3c92433a172d9c3142128074f2b" frameborder=0></iframe>

![KDE plot of Gamma Ray data from multiple wells generated using Python’s matplotlib library. Image created by the author.](https://cdn-images-1.medium.com/max/2000/1*737Oz3bpOMqDBsE457VEGA.png)

## Summary

In this article we have covered how to load multiple LAS files by searching a directory for all files with a .las extension and concatenate them into a single [pandas dataframe](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html). Once we have this data in a dataframe, we can easily call upon matplotlib and seaborn to make quick and easy to understand visualizations of the data.

***Thanks for reading!***

*If you have found this article useful, please feel free to check out my other articles looking at various aspects of Python and well log data. You can also find my code used in this article and others at [**GitHub](https://github.com/andymcdgeo).***

*If you want to get in touch you can find me on [**LinkedIn](https://www.linkedin.com/in/andymcdonaldgeo/) **or at my [**website](http://andymcdonald.scot/)**.*

*Interested in learning more about python and well log data or petrophysics? Follow me on [**Medium](https://andymcdonaldgeo.medium.com/)**.*
