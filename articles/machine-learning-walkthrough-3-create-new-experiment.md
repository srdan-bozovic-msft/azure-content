<properties title="" pageTitle="Step 3: Create a new Machine Learning experiment | Azure" description="Step 3: Create a new training experiment in Azure Machine Learning Studio" metaKeywords="" services="machine-learning" solutions="" documentationCenter="" authors="Garyericson" manager="paulettm" editor="cgronlun" videoId="" scriptId=""/>

<tags ms.service="machine-learning" ms.workload="data-services" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="09/02/2014" ms.author="garye" />


This is the third step of the walkthrough, [Developing a Predictive Solution with Azure ML][develop]:

[develop]: ../machine-learning-walkthrough-develop-predictive-solution/


1.	[Create an ML workspace][create-workspace]
2.	[Upload existing data][upload-data]
3.	**Create a new experiment**
4.	[Train and evaluate the models][train-models]
5.	[Publish the web service][publish]
6.	[Access the web service][access-ws]

[create-workspace]: ../machine-learning-walkthrough-1-create-ml-workspace/
[upload-data]: ../machine-learning-walkthrough-2-upload-data/
[create-new]: ../machine-learning-walkthrough-3-create-new-experiment/
[train-models]: ../machine-learning-walkthrough-4-train-and-evaluate-models/
[publish]: ../machine-learning-walkthrough-5-publish-web-service/
[access-ws]: ../machine-learning-walkthrough-6-access-web-service/

----------

# Step 3: Create a new Azure Machine Learning experiment

We need to create a new experiment in ML Studio that uses the dataset we uploaded.  

1.	In ML Studio, click **+NEW** at the bottom of the window.
2.	Select **EXPERIMENT**.
3.	In the module palette to the left of the experiment canvas, expand **Saved Datasets**.
4.	Find the dataset you created and drag it onto the canvas. You can also find the dataset by entering the name in the **Search** box above the palette.  

##Prepare the data
You can view the first 100 rows of the data and some statistical information for the whole dataset by right-clicking the output port of the dataset and selecting **Visualize**. Notice that ML Studio has already identified the data type for each column. It has also given generic headings to the columns because the data file did not come with column headings.  

Column headings are not essential, but they will make it easier to work with the data in the model. Also, when we eventually publish this model in a web service, the headings will help identify the columns to the user of the service.  

We can add column headings using the **Metadata Editor** module.  

1.	In the module palette, type "metadata" in the **Search** box. You'll see **Metadata Editor** in the module list.
2.	Click and drag the **Metadata Editor** module onto the canvas and drop it below the dataset.
3.	Connect the dataset to the **Metadata Editor**: click the output port of the dataset, drag to the input port of **Metadata Editor**, then release the mouse button. The dataset and module will remain connected even if you move either around on the canvas.
4.	In the **Properties** pane to the right of the canvas, click **Launch column selector**.
5.	Verify that "All columns" is selected in the **Begins With** field, remove any rows, and click **OK**. This directs the **Metadata Editor** to operate on all columns of data.
6.	For the **New column name** parameter, enter a list of names for the 21 columns in the dataset, separated by commas and in column order. You can obtain the columns names from the dataset documentation on the UCI website, or for convenience you can copy and paste the following:  

	Status of checking account, Duration in months, Credit history, Purpose, Credit amount, Savings account/bond, Present employment since, Installment rate in percentage of disposable income, Personal status and sex, Other debtors, Present residence since, Property, Age in years, Other installment plans, Housing, Number of existing credits, Job, Number of people providing maintenance for, Telephone, Foreign worker, Credit risk  

The Properties pane will look like this:

![Properties for Metadata Editor][1] 

>Tip - If you want to verify the column headings, run the experiment (click **RUN** below the experiment canvas), right-click the output port of the **Metadata Editor** module, and select **Visualize**. You can view the output of any module in the same way to view the progress of the data through the experiment.

The experiment should now look something like this:  

![Adding Metadata Editor][2]
 
##Create training and test datasets
The next step of the experiment is to generate separate datasets that will be used for training and testing our model. To do this, we use the **Split** module.  

1.	Find the **Split** module, drag it onto the canvas, and connect it to the last **Metadata Editor** module.
2.	By default, the split ratio is 0.5 and the **Randomized split** parameter is set. This means that a random half of the data will be output through one port of the **Split** module, and half out the other. You can adjust these, as well as the **Random seed** parameter, to change the split between training and scoring data. For this example we'll leave them as-is.
	>Tip - The split ratio essentially determines how much of the data is output through the left output port. For instance, if you set the ratio to 0.7, then 70% of the data is output through the left port and 30% through the right port.  
	
We can use the outputs of the **Split** module however we like, but let's choose to use the left output as training data and the right output as scoring data.  

As mentioned on the UCI website, the cost of misclassifying a high credit risk as low is 5 times larger than the cost of misclassifying a low credit risk as high. To account for this, we'll generate a new dataset that reflects this cost function. In the new dataset, each high example is replicated 5 times, while each low example is not replicated.   

We can do this replication using R code:  

1.	Find and drag the **Execute R Script** module onto the experiment canvas and connect it to the left output port of the **Split** module.
2.	In the **Properties** pane, delete the default text in the **Script** parameter and enter this script: 

		dataset1 <- maml.mapInputPort(1)
		data.set<-dataset1[dataset1[,21]==1,]
		pos<-dataset1[dataset1[,21]==2,]
		for (i in 1:5) data.set<-rbind(data.set,pos)
		maml.mapOutputPort("data.set")


We need to do this same replication operation for each output of the **Split** module so that the training and scoring data have the same cost adjustment.

1.	Right-click the **Execute R Script** module and select **Copy**.
2.	Right-click the experiment canvas and select **Paste**.
3.	Connect this **Execute R Script** module to the right output port of the **Split** module.  

>Tip - The copy of the Execute R Script module contains the same script as the original module. When you copy and paste a module on the canvas, the copy retains all the properties of the original.  
>
Our experiment now looks something like this:
 
![Adding Split module and R scripts][3]

**Next: [Train and evaluate the models][train-models]**


[1]: ./media/machine-learning-walkthrough-3-create-new-experiment/create1.png
[2]: ./media/machine-learning-walkthrough-3-create-new-experiment/create2.png
[3]: ./media/machine-learning-walkthrough-3-create-new-experiment/create3.png
