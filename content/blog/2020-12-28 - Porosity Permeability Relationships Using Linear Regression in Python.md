---
title: "Porosity-Permeability Relationships Using Linear Regression in Python"
date: "2020-12-28"
draft: false
categories: ["Petrophysics", "Python"]
tags: ["petrophysics", "python", "porosity", "permeability", "crossplots"]
featured_image: "/images/featured/2020-12-28- Porosity Permeability Linear Regression Crossplots.jpg"
---

<!-- ## Porosity-Permeability Relationships Using Linear Regression in Python

### A short guide on applying a linear regression in Python to semi-log data

![Photo by [Ekaterina Novitskaya](https://unsplash.com/@catrionaobrian?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/10944/0*iZcMgI1f7AfYNUPv) -->

Core data analysis is a key component in the evaluation of a field or discovery, as it provides direct samples of the geological formations in the subsurface over the interval of interest. It is often considered the ‘ground truth’ by many and is used as a reference for calibrating well log measurements and petrophysical analysis. Core data is expensive to obtain and not acquired on every well at every depth. Instead, it may be acquired at discrete intervals on a small number of wells within a field and then used as a reference for other wells.

Once the core data has been extracted from the well it is taken to a lab to be analysed. Along the length of the retrieved core sample a number of measurements are made. Two of which are porosity and permeability, both key components of a petrophysical analysis.

* **Porosity** is defined as the volume of space between the solid grains relative to the total rock volume. It provides an indication of the potential storage space for hydrocarbons.

* **Permeability** provides an indication of how easy fluids can flow through the rock.

Porosity is a key control on permeability, with larger pores resulting in wider pathways for the reservoir fluids to flow through.

Well logging tools do not provide a direct measurement for permeability and therefore it has to be inferred through relationships with core data from the same field or well, or from empirically derived equations.

One common method is to plot porosity (on a linear scale) against permeability (on a logarithmic scale) and observe the trend. From this, a regression can be applied to the porosity permeability (poro-perm) crossplot to derive an equation, which can subsequently be used to predict a continuous permeability from a computed porosity in any well.

![Porosity vs Permeability Crossplot with Python Statsmodels prediction (red line).](https://cdn-images-1.medium.com/max/2000/1*YzIuv0Ct5Vdyq_QXN0kzIA.png)

In this article, I will cover how carry out a porosity-permeability regression using two methods within Python: [numpy’s polyfit](https://numpy.org/doc/stable/reference/generated/numpy.polyfit.html) and [statsmodels Ordinary Least Squares regression](https://www.google.com/search?client=safari&rls=en&q=ols+statsmodel&ie=UTF-8&oe=UTF-8).

The notebook for this article can be found on my Python and Petrophysics Github series which can accessed at the link below:

[https://github.com/andymcdgeo/Petrophysics-Python-Series](https://github.com/andymcdgeo/Petrophysics-Python-Series)

Additionally, a list of previous articles, notebooks and blog posts can be found on my website here:

[http://andymcdonald.scot/python-and-petrophysics](http://andymcdonald.scot/python-and-petrophysics)

## Importing Libraries and Core Data

To begin, we will import a number of common libraries before we start working with the actual data. For this article we will be using [pandas](https://pandas.pydata.org), [matplotlib](https://matplotlib.org) and [numpy](https://numpy.org/doc/stable/index.html). These three libraries allow us to load, work with and visualise our data.

    import pandas as pd
    import matplotlib.pyplot as plt
    import numpy as np

The dataset we are using comes from the publicly available [Equinor Volve Field dataset](https://www.equinor.com/en/what-we-do/norwegian-continental-shelf-platforms/volve.html) released in 2018. The files used in this tutorial are from 15/9- 19A which contain full regular core analysis data and well log data. To load this data in we can use pd.read_csv and pass in the file name.

This dataset has already been depth aligned to well log data, so no adjustments to the sample depth are required.

When core slabs are analysed, a limited number of measurements are made at irregular intervals. In some cases, measurements may not be possible, for example in really tight (low permeability) sections. As a result, we can tell pandas to load any missing values / blank cells as Not a Number (NaN) by adding the argument na_values=' ' .

    core_data = pd.read_csv("Data/15_9-19A-CORE.csv", na_values=' ')

Once we have the data loaded, we can view the details of what is in it by calling upon the .head() and .describe() methods.

The .head() method returns the first five rows of the dataframe and the header row.

    core_data.head()

![Results from the head function of our pandas dataframe. This shows us the first 5 rows of data and column headers.](https://cdn-images-1.medium.com/max/2000/1*GN9uf_BtFG9RhI0CikFxTA.png)

The .describe() method returns useful statistics about the numeric data contained within the dataframe such as the mean, standard deviation, maximum and minimum values.

    core_data.describe()

![Results from the describe function of pandas on our core dataframe.](https://cdn-images-1.medium.com/max/2000/1*DjUfdM1deVJ2aGlITuJNkA.png)

## Plotting Porosity vs Permeability

Using our core_data dataframe we can simply and quickly plot our data by adding .plot to the end of our dataframe and supplying some arguments. In this case we want a scatter plot (also known in petrophysics as a crossplot), with the x-axis as CPOR — Core Porosity and the y-axis as CKH — Core Permeability.

    core_data.plot(kind="scatter", x="CPOR", y="CKH")

![Simple linear-linear scatterplot of porosity vs permeability using matplotlib and pandas.](https://cdn-images-1.medium.com/max/3600/1*-_qrQvehwVIpreVrD09shg.png)

From this scatter plot we notice that there is a large concentration of points at low permeabilities with a few points at the higher end. We can tidy up our plot by converting the y axis to a logaritmic scale and adding a grid. This generates the the poro-perm crossplot that we are familiar with in petrophysics.

    core_data.plot(kind="scatter", x="CPOR", y="CKH")
    plt.yscale('log')
    plt.grid(True)

![Log-Linear scatterplot of porosity vs permeability using matplotlib and pandas.](https://cdn-images-1.medium.com/max/3600/1*u2TCrOjhOqgqw-nEoJYAxg.png)

We can agree that this looks much better now. We can further tidy up the plot by:

* Switching to matplotlib for making out plot

* Adding labels by using ax.set_ylabel() and ax.set_xlabel()

* Setting ranges for the axis using ax.axis([0,40, 0.01, 100000)

* Making the y-axis values easier to read by converting the exponential notation to full numbers. This is done using FuncFormatter from matplotlib and setting up a simple for loop

    from matplotlib.ticker import FuncFormatter
    fig, ax = plt.subplots()

    ax.axis([0, 40, 0.01, 100000])
    ax.plot(core_data['CPOR'], core_data['CKHG'], 'bo')
    ax.set_yscale('log')
    ax.grid(True)
    ax.set_ylabel('Core Perm (mD)')
    ax.set_xlabel('Core Porosity (%)')

    #Format the axes so that they show whole numbers
    for axis in [ax.yaxis, ax.xaxis]:
        formatter = FuncFormatter(lambda y, _: '{:.16g}'.format(y))
        axis.set_major_formatter(formatter)

![Formatted log-linear scatter plot of porosity vs permeability using matplotlib.](https://cdn-images-1.medium.com/max/3600/1*-732VIjNSubs-NoDFqJM4g.png)

Isn’t that much better than the previous plot? We can now use this nicer looking plot within a petrophysical report or passing to other subsurface people within the team.

## Deriving Relationship Between Porosity and Permeability

There are two ways that we can carry out a poro-perm regression on our data:

* Using numpy’s polyfit function

* Applying a regression using the statsmodels library

Before we explore each option, we first have to create a copy of our dataframe and remove the null rows. Carrying out the regression with NaN values can result in errors.

    poro_perm = core_data[['CPOR', 'CKHG']].copy()

Once it has been copied we can then drop the NaNs using dropna(). Including the argument inplace=True tells the method to replace the values in place rather than returning a copy of the dataframe.

    poro_perm.dropna(inplace=True)

### Numpy polyfit()

The simplest option for applying a linear regression through the data is using the polynomial fit function from numpy. This returns an array of co-efficients. As we are wanting to use a linear fit we can specify a value of 1 at the end of the function. This tells the function we want a first degree polynomial.

Also, are we are dealing with permeability data in the logarithmic scale, we need to take the logarithm of the values using np.log10.

    poro_perm_polyfit = np.polyfit(core_data['CPOR'], np.log10(core_data['CKH']), 1)

When we check the value of poor_perm_polyfit, we return:array([0.17428705, -1.55607816])

The first value is our slope and the second is our y-intercept.

Polyfit doesn’t give us much more information about the regression such as the co-efficient of determination (R-squared). For this we need to look at another model.

### Statsmodels Linear Regression

The second option for generating a poro-perm linear regression is to use the Ordinary Least Squares (OLS) method from the statsmodels library.

First we need to import the library and create our data. We will assign our x value as Core Porosity (CPOR) and our y value as the log10 of Core Permeability (CKH). The y value will be the one we are aiming to build our prediction model from.
>  With the statsmodel OLS we need to add a constant column to our data as an intercept is not included by default unless we are using formulas. See [here](https://www.statsmodels.org/devel/generated/statsmodels.regression.linear_model.OLS.html) for the documentation.

    import statsmodels.api as sm

    x = core_data['CPOR']
    x = sm.add_constant(x)
    y = np.log10(core_data['CKHG'])

We can confirm the values of x by calling upon it and in Jupyter it will return a dataframe as seen here:

![Our x variables: core porosity (CPOR) and a constant column of 1s.](https://cdn-images-1.medium.com/max/2000/1*h1hWFdhosCC3BFw7SzPP1g.png)

The next step is to build and fit our model. With the OLS method, we can supply an argument for missing values. In this example I have set it to drop. This will remove or drop the missing values from the data.

    model = sm.OLS(y, x, missing='drop')
    results = model.fit()

Once we have fitted the model, we can view a full summary of the regression by calling upon .summary()

    results.summary()

Which returns a nicely formatted table like the one below and includes key statistics as the R-squared and standard error.

![OLS summary from statsmodels for a linear regression.](https://cdn-images-1.medium.com/max/2000/1*NMAwycT8nl-qv2VJsaGGSA.png)

We can also obtain the key parameters: slope and intercept, by calling upon results.params.

![Slope and intercept values from our linear regression.](https://cdn-images-1.medium.com/max/2000/1*sjFJ6pEakHlzre9SSRw5Eg.png)

If we want to access one of the parameters, for example the slope or constant for the CPOR value, we can access it like a list:

    results.params[1]

Which returns a value of: 0.174287

We can then piece together the equation we will use to predict our permeability:

![Relationship for permeability and porosity from Ordinary Least Squares Regression.](https://cdn-images-1.medium.com/max/2000/1*7u_jTo-3Vs_xIS_naSIPLQ.png)

Finally, we can take our equation and apply it to our scatter plot using the line:

    ax.semilogy(core_data['CPOR'], 10**(results.params[1] * core_data['CPOR'] + results.params[0]), 'r-')

The whole code for the scatter plot:

    from matplotlib.ticker import FuncFormatter
    fig, ax = plt.subplots()

    ax.axis([0, 30, 0.01, 100000])
    ax.semilogy(core_data['CPOR'], core_data['CKHG'], 'bo')

    ax.grid(True)
    ax.set_ylabel('Core Perm (mD)')
    ax.set_xlabel('Core Porosity (%)')

    ax.semilogy(core_data['CPOR'], 10**(results.params[1] * core_data['CPOR'] + results.params[0]), 'r-')

    #Format the axes so that they show whole numbers
    for axis in [ax.yaxis, ax.xaxis]:
        formatter = FuncFormatter(lambda y, _: '{:.16g}'.format(y))
        axis.set_major_formatter(formatter)

![Porosity vs Permeability Crossplot with Python Statsmodels prediction (red line).](https://cdn-images-1.medium.com/max/2000/1*YzIuv0Ct5Vdyq_QXN0kzIA.png)

## Predicting a Continuous Permeability from Log Porosity

Now that we have our equation and we are happy with the results, we can apply this to our log porosity to generate a continuous permeability curve.

First, we need to load in the well log data for this well:

    well = pd.read_csv('Data/15_9-19.csv', skiprows=[1])

And then apply our derived formula to the PHIT curve:

    well['PERM']= 10**(results.params[1] * (well['PHIT']*100) + results.params[0])

When we check the well header using well.head()we can see our newly created curve at the end of the dataframe.

## Visualising the Final Predicted Curve

The final step in our workflow is to plot the PHIT curve and the predicted permeability curve on a log plot alongside the core measurements:

    fig, ax = plt.subplots(figsize=(5,10))
    ax1 = plt.subplot2grid((1,2), (0,0), rowspan=1, colspan = 1)
    ax2 = plt.subplot2grid((1,2), (0,1), rowspan=1, colspan = 1, sharey = ax1)

    # Porosity track
    ax1.plot(core_data["CPOR"]/100, core_data['DEPTH'], color = "black", marker='.', linewidth=0)
    ax1.plot(well['PHIT'], well['DEPTH'], color ='blue', linewidth=0.5)
    ax1.set_xlabel("Porosity")
    ax1.set_xlim(0.5, 0)
    ax1.xaxis.label.set_color("black")
    ax1.tick_params(axis='x', colors="black")
    ax1.spines["top"].set_edgecolor("black")
    ax1.set_xticks([0.5,  0.25, 0])

    # Permeability track
    ax2.plot(core_data["CKHG"], core_data['DEPTH'], color = "black", marker='.', linewidth=0)
    ax2.plot(well['PERM'], well['DEPTH'], color ='blue', linewidth=0.5)
    ax2.set_xlabel("Permeability")
    ax2.set_xlim(0.1, 100000)
    ax2.xaxis.label.set_color("black")
    ax2.tick_params(axis='x', colors="black")
    ax2.spines["top"].set_edgecolor("black")
    ax2.set_xticks([0.01, 1, 10, 100, 10000])
    ax2.semilogx()

    # Common functions for setting up the plot can be extracted into
    # a for loop. This saves repeating code.
    for ax in [ax1, ax2]:
        ax.set_ylim(4025, 3825)
        ax.grid(which='major', color='lightgrey', linestyle='-')
        ax.xaxis.set_ticks_position("top")
        ax.xaxis.set_label_position("top")
        
    # Removes the y axis labels on the second track
    for ax in [ax2]:
        plt.setp(ax.get_yticklabels(), visible = False)
        
    plt.tight_layout()
    fig.subplots_adjust(wspace = 0.3)

This generates a simple two track log plot with our core measurements represented by black dots and our continuous curves by blue lines.

![Final permeability prediction from core measurements using Python statsmodels library and ordinary least squares regression.](https://cdn-images-1.medium.com/max/2000/1*GWk8hBBwMOBYoUE8B0xkXQ.png)

As seen in track 2, our predicted permeability from a simple linear regression tracks the core permeability reasonably well. However, between about 3860 and 3875, our prediction reads lower than the actual core measurements. Also, it becomes harder to visualise the correlation at the lower interval to the more thinly bedded nature of the geology.

## Conclusion

In this walkthrough, we have covered what core porosity and permeability area and how we can predict the latter from the former to generate an equation that can be used to predict a continuous curve. This can subsequently be used in geological models or reservoir simulations.

As noted at the end, there are a few small mismatches. These would benefit from further investigation and potentially further modelling either by refining the regression or by applying another machine learning model.
