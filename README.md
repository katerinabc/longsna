---
# Longitudinal Social Network Analysis


## Analytical Models


### Stochastic Actor-oriented Model (SAOM)
The **stochastic actor-oriented model** was developed by [Tom Snijders and colleagues](https://www.stats.ox.ac.uk/~snijders/siena/). Its purpose is to represent network dynamics based on observed longitudinal data, and to draw conclusions about the analyzed populations based on the model. Variants of SAOM for multiplex data (co-evolution of edges) and for modeling networks collected in different groups (e.g., different classrooms) exists. The following assumptions hold for SAOM models:

* Time *t* is continuous. While the parameter estimation assumes that data is observed at two or more discrete points in time (*network panel wave*), the model assumes that between the observed data collection periods, unobserved mini-steps are happening. This allows for explaining how three people who at time 0 were not connected, suddenly, at time 1 form a closed triangle. This closed triangle did not appear out of nowhere but is the result of three mini steps which happened while you were not observing your participants. 
* The observed network is the consequences of previous networks. This means that all information that is contained in an earlier network (t-1) determine the probability with which ties are present/absent at the next stage (t). Factors that occurred even further back in time (e.g. t-2) have no impact on the network at time t. This is called a *Markov process*.
* The person sending the tie (*ego*, denoted as *i*) decides if he/she wants to be connected to another person (*alter*, denoted as *j*). This excludes any relationships that are based on negotiations. Network formation is thus *actor-based*.
* At any moment in time, only one outgoing tie can change. Hence, *ties change one by one*. This makes the network easier to model. 
* The rate at which ties changes depends on an actor's and her/his alters network position and covariates.
* It is assumed that all actors have the same propensity to change their ties. In other words, they have the same objective function. Heterogeneity is added to the model by including network effects and covariates. However, the underlying assumption is that actors only evaluates the impact of their decision on their network position, and do not consider subsequent decisions by other actors. Hence, actors do not evaluate the responses from other actors.  

### Temporal Exponential Random Graph Model (TERGM)
The **temporal exponential random graph model** was developed by [Steve Hanneke and colleagues](http://repository.cmu.edu/cgi/viewcontent.cgi?article=1232&context=machine_learning) 

and is an extension to the simple exponential random graph model (ERGM). In simple terms, ERGM is a logistic regression that calculates the probability that an edge is present or absent based on the network structure and covariates. TERGM is an extension of ERGM. 

TERGM and SAOM are pretty similar. Below a description of their differences to aid when one method is more appropriate than the other:
* In contrast to SAOM, TERMG does not make any assumptions about the agency of actors. While the mathematics behind the model make no claim about actor agency, the way in which the model is created has an actor-centric flavor. 
* TERGM does not consider the mini-steps that might occur between *t* and *t+1*. It accepts that edges which did not exist at *t* but exists at *t+1* might have occurred sequentially or simultaneously. 
* SAOM purposefully models the change in networks between time periods. TERGM focuses on modeling the network. The temporal dimension is included by adding a *memory* term in the equation as a control variable. This memory term references the previously observed network. 

While from a theoretical perspective, SAOM outperforms TERGM, testing of the two models using real data showed that TERGM more accurately predicted edges. 

 variants for "multiplex", "multi-relational", or "multi-level" ERGMs as well (Wang et al. 2013), but they have not yet been extended to the temporal case.

### Relational Event Model (REM)
The **relational event model** was developed by [Carter T Butts and colleagues](http://journals.sagepub.com/doi/10.1111/j.1467-9531.2008.00203.x). It focuses on behavioral interactions, which are defined as *discrete events* directed at a person or a group of people. It assumes that past interactions create the context for future interactions. This means that every action that occurs depends on the action which occurred right before it. The following assumptions apply:
* At each time point *t* only one event can occur.
* A current event *t* is not influenced by the realization of future events *t+1* (reverse causality).
* A current event *t* is not influenced by the non-occurrence of an event that could have appended at *t-1*. 
The consequence of these assumptions is that REM is less suited for situations in which individuals have a strategic orientation, are able to engage in forward-looking behavior, or have time for observation and reflection.  

## Running a SAOM

### Background

#### Context
Below is a short example of the steps necessary to run a SAOM. The data represents interaction between team members in several student teams. Each team consisted of around five master students and worked on a project for a client. The network was collected at the beginning of the course (*t0*), 3 weeks into the course (*t1*), and at the end of the course in week 7, before the grades were published (*t2*). More information about the context is available in [Unbundling Information Exchange in Ad Hoc Project Teams](https://www.researchgate.net/project/May-I-ask-you-The-influence-of-individual-dyadic-and-network-factors-on-the-emergence-of-information-exchange-in-teams).

#### Variables
The variables we collected were *information retrieval* and *information allocation* (dependent variables), awareness of team member's expertise (*knowing*, independent variable), the importance team member's attach to each other's expertise (*valuing*, independent variable), how adaptive individuals are (*adaptive expertise*, independent variable), their emotional attachment to their group (*social identity*, independent variable), and background variables (*gender*, *age*, *nationality*, *specialization*, control variable).

#### What we did
We analyzed the data using a hierarchical stochastic actor oriented model using Bayesian regression. We made this choice due to the small size of network. While we had two dependent variables, we did not have enough data to calculate a co-evolution model, thus we could not estimate how information retrieval and information allocation influence each other. In place of this we calculated two models, one for information allocation and one for information retrieval. 

### SAOM Steps
For this workshop we will be focusing on modeling information retrieval for three teams using only valuing, and adaptive expertise as dependent variables. Running the complete model, as conducted in the linked study, was computational intensive. 

In general, the steps are: 
1. Import the data
2. Specify independent, dependent variables
3. Select the effects and specify the model
4. Test goodness of fit


#### Loading and preparing the data
We will first load the required packages and set our working directory.

If you don't have the packages installed, run this line. If this fails, check the error message. Did a package that is needed for RSienaTest to run, fail to install?
If this fails with a non-zero exist status and you have a mac, do you have the development tools (xcode) installed?

```{r}
#install.packages("RUnit")
#install.packages("RSienaTest", repos="http://R-Forge.R-project.org", dependencies=T)
```

Use `setwd` to set the path to your working directory. For example, `setwd("~/Dropbox/longitudinal_sna_asc_workshop/data")`. In windows this would look like `setwd("c:/Dropbox/longitudinal_sna_asc_workshop/data")`. If you are unsure about your working directory you can type `getwd` in the console. 
```{r}
library(RSienaTest)
```

We begin with loading the data. A couple of words about the code to import the data. All the data about one variable for one team is assigned to one object (`variable name <- as.matrix(...)`). `read.csv` reads the data about one team. I have for each team one *txt* file. Therefore, I need to skip some rows (`skip = 4`) and tell the program how many rows to import (`nrows = 4`). I also need to add row names (`row.names = 1`) and column names (`col.names = c(name1, name2, etc.)`). I'm ok with having row names and column names that have the same names (`check.names=FALSE`). While not important, I'm telling R to convert string variables into dummy variables (`stringsAsFactors =TRUE`). That is not important as I don't have any string variables. The only string are the header variables, and I indicated them. All data about one team is imported in one list (`list(....)`) to make the cleaning easier. 

**Network variables**
```{r}
adulteduc<-list(
#adulteduc t1
ae_val1<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-5.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Daniel", "Liga", "Shengye", "Valentina") , skip=4, nrows=4, check.names=FALSE), stringsAsFactors=TRUE),
ae_is1<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-5.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Daniel", "Liga", "Shengye", "Valentina") , skip=14, nrows=4, check.names=FALSE), stringsAsFactors=TRUE),
#aduleteduc t2
ae_val2<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-5.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Daniel", "Liga", "Shengye", "Valentina") , skip=24, nrows=4, check.names=FALSE), stringsAsFactors=TRUE),
ae_is2<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-5.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Daniel", "Liga", "Shengye", "Valentina") , skip=34, nrows=4, check.names=FALSE), stringsAsFactors=TRUE),
#aduleteduc t3
ae_val3<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-5.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Daniel", "Liga", "Shengye", "Valentina") , skip=44, nrows=4, check.names=FALSE), stringsAsFactors=TRUE),
ae_is3<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-5.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Daniel", "Liga", "Shengye", "Valentina") , skip=54, nrows=4, check.names=FALSE), stringsAsFactors=TRUE)
)

alpheus<-list(
#Alpehus t1
alp_val1<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-6.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Sarah", "Harmke", "Sybil", "Sanne") , skip=4, nrows=4, check.names=FALSE), stringsAsFactors=TRUE),
alp_is1<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-6.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Sarah", "Harmke", "Sybil", "Sanne") , skip=14, nrows=4, check.names=FALSE), stringsAsFactors=TRUE),
#Alpehus t2
alp_val2<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-6.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Sarah", "Harmke", "Sybil", "Sanne") , skip=24, nrows=4, check.names=FALSE), stringsAsFactors=TRUE),
alp_is2<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-6.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Sarah", "Harmke", "Sybil", "Sanne") , skip=34, nrows=4, check.names=FALSE), stringsAsFactors=TRUE),
#Alpehus t3
alp_val3<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-6.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Sarah", "Harmke", "Sybil", "Sanne") , skip=44, nrows=4, check.names=FALSE), stringsAsFactors=TRUE),
alp_is3<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-6.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Sarah", "Harmke", "Sybil", "Sanne") , skip=54, nrows=4, check.names=FALSE), stringsAsFactors=TRUE)
)

hht<-list(
#hht T1
hht_val1<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-7.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Anja","Johanna","Erika","Esmina","Saulius") , skip=5, nrows=5, check.names=FALSE), stringsAsFactors=TRUE),
hht_is1<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-7.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Anja","Johanna","Erika","Esmina","Saulius") , skip=17, nrows=5, check.names=FALSE), stringsAsFactors=TRUE),
#hht T2
hht_val2<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-7.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Anja","Johanna","Erika","Esmina","Saulius") , skip=29, nrows=5, check.names=FALSE), stringsAsFactors=TRUE),
hht_is2<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-7.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Anja","Johanna","Erika","Esmina","Saulius") , skip=41, nrows=5, check.names=FALSE), stringsAsFactors=TRUE),
#hht T3
hht_val3<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-7.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Anja","Johanna","Erika","Esmina","Saulius") , skip=53, nrows=5, check.names=FALSE), stringsAsFactors=TRUE),
hht_is3<-as.matrix(read.csv("data/ae_teamdynamics_MASTER_UCINET-7.txt",na.strings=9, sep = ";", dec=",", row.names=1, col.names=c("row.nameS", "Anja","Johanna","Erika","Esmina","Saulius") , skip=65, nrows=5, check.names=FALSE), stringsAsFactors=TRUE)
)
```


**Actor attributes**
```{r}
attributes<-read.csv("data/attributes_article.csv", header=TRUE, row.names=1)
ae_att<-subset(attributes, team =="adult education")
alp_att<-subset(attributes, team =="alpheus")
hht_att<-subset(attributes, team =="hht")
```


Now that we have the data we need to prepare it. The network variables are valued ranging from 2 (*rarely* for information retrieval and *strongly disagree* for valuing) to 6 (*very frequent* for information retrieval and *strongly agree* for valuing). While it is possible to do SAOM with valued data I did the simpler route and dichotomized my networks.

Below we define a simple function *dicoGT4* that takes a matrix and changes all values above 4 to 1 and the other values to 0. We apply this function to our networks. As we have stored them in a list, this is done in 1 line, instead of 18 lines (3 (for each team) x 3 (for each wave of data) X 2 (for each network variable))
```{r}
dichoGT4<-function(m){(m>4)+ 0}
alpheusgt4<-lapply(alpheus, dichoGT4)
adulteducgt4<-lapply(adulteduc, dichoGT4)
hhtgt4<-lapply(hht, dichoGT4)
```

#### Creating SAOM variables
Now we have our variables and we are ready to create *siena* variables. These are necessary to tell the program *RSiena* which variables are dependent (*sienanet*) and which are independent. Independent variables can be dyadic covariates (*varDyadCovar*) or individual covariates (*coCovar*). The covariates can be changing or static. In my case they were static, meaning that they did not change between data collection periods. Once this is done, the last step is to put it all together and tell the program which variables belong to which team.

Remember that we loaded our variables for each team as a list. When you have a list, you need to include two square brackets *[[ ]]* to get to the content of the list. Imagine a list like an excel file with each list item as an excel sheet. The *dim* variable indicates how many actors are in the network (5) and how many time periods (3). 
```{r}
ae_is <- sienaNet(array(c(adulteducgt4[[2]],adulteducgt4[[4]],adulteducgt4[[6]]), dim=c(4,4,3)), allowOnly=FALSE) 
alp_is <-sienaNet(array(c(alpheusgt4[[2]],alpheusgt4[[4]],alpheusgt4[[6]]), dim=c(4,4,3)), allowOnly=FALSE)
hht_is <-sienaNet(array(c(hhtgt4[[2]],hhtgt4[[4]],hhtgt4[[6]]), dim=c(5,5,3)), allowOnly=FALSE)

ae_val <- varDyadCovar(array(c(adulteduc[[1]], adulteduc[[3]]), dim=c(4,4,2)))
alp_val <-varDyadCovar(array(c(alpheus[[1]], alpheus[[3]]), dim=c(4,4,2)))
hht_val <-varDyadCovar(array(c(hht[[1]], hht[[3]]), dim=c(5,5,2)))

ae_ae<-coCovar(ae_att$aev2)
alp_ae<-coCovar(alp_att$aev2)
hht_ae<-coCovar(hht_att$aev2)
```

Let's create the groups!
```{r}
group.1<- sienaDataCreate(IS = ae_is, 
                          val=ae_val,
                          ae = ae_ae
                          )

group.2<- sienaDataCreate(IS = alp_is, 
                          val=alp_val,
                          ae = alp_ae
                          )

group.3<- sienaDataCreate(IS = hht_is, 
                          val = hht_val,
                          ae = hht_ae)

```


#### Running a basic SAOM Model
We will now begin with running a basic model. This will be our null model and only contains the most basic effects you can think off. To have good convergence, use theory and exploration of your data to consider what network structures could explain your network. Similar to running an ERGM this is a bit of try and error (or an art as I heard other people say). You should have a look at the extensive documentation of network effects. Of course, you can also create your own network effects, interactions between variables. 

*sienaGroupCreate* creates the groups. It just puts them together. The model we are using will have a random intercept for each group, but a fixed slope.
*getEffects* creates an object with the theoretical network structures for our groups.
*coevalgo*
The file *coeveffect.html* provides an overview of all the effects and their naming. It contains more effects that are available to you right now, as it is based on the complete data set. 
```{r}
ir <- sienaGroupCreate(list(group.1, group.2, group.3))
ir_effect <- getEffects(ir)
ir_algo<-sienaAlgorithmCreate(projname = "InfoRetrieval", maxlike=TRUE, mult=6)
#get initial description
print01Report(ir, modelname = 'InfoRetrieval')
```

The last line in the code above prints a report about your data. It contains some basic statistics (e.g., mean). 

We will begin by adding some variables to our model. This is done by adding effects to our effect object *ir_effect*. We begin by changing reciprocity (*recip*) from a fixed effect to a random effect. 

```{r}
ir_effect <- setEffect(ir_effect, recip, random=T)
```
The model is based on Bayesian analysis. This is beyond the scope of this workshop. Focusing on *but-what-do-I-do-with* you can leave the values for *fix*, *test*, *initialValue*, and *parm* untouched (for now). You can type `?setEffect` to get information about the different options. 

Then we add the variable *valuing* by stating its name (*interaction1 = "val"*) and that we want to know how it impacts the creation, maintenance, or dissolution of ties (*type = "eval"*). You can also only test the impact of a variable on the evaluation (**XXX**) and/or endowment (**XX**) of ties. *X* indicates the network for which the effects are included. 
We do the same for the variable *adaptive expertise*. Here, we say that this variable impacts the position of the ego, and hence add *egoX* and not *X*. 
Finally, we are checking if all is ok b asking R to print the effects. 

```{r}
ir_effect <- includeEffects(ir_effect, X, interaction1= "val", type="eval")
ir_effect <- includeEffects(ir_effect,egoX, interaction1= "ae", type="eval")

print(ir_effect, includeRandoms=T)
```
The output shows the variables we are including. We assume that density and reciprocity vary between groups. The effect of valuing and adaptive expertise is the same across teams. 

Now that the model is specified we can run it. This is done by using `sienaBayes`. You can change a lot of the parameters for running the algorithms. Higher numbers for `nwarm`, `nmain`, `nrunMHBatches` will give you more accurate results, but will also increase the computation times. So, be careful how you change them. `nbrNodes` indicates the number of processes to use on your CPU. 

The first line `fit1 <- sienaBayes(...)` contains all the information to run the model. In the following line we are asking R to return the model and then we save it all in a text file called *results_of_model.txt*. This is done with an opening statement `sink(filename)` and a closing statement (`sink()`). We are appending the current results to the file. If you are running several models and want to keep track of the results this might be helpful. Without `append =T` you will overwrite the file. 

The parameters used in this sienaBayes call are not the ones I used in my analysis. All the values are lowered for the workshop. The original values I used were `sienaBayes(coevalgo, data=coev, effects=coeveffect, nwarm=100, nmain=50000, nrunMHBatches=20, initgainGroupwise=0, initgainGlobal=0, silentstart=TRUE, nbrNodes=18)`. Running the model below takes a short time (64.259 seconds). Running the same model using all teams takes a bit longer (1481.995 seconds). 

```{r}
fit1 <- sienaBayes(ir_algo, data=ir, effects=ir_effect, nwarm=10, nmain=25, nrunMHBatches=10)
```

Of course, we want to see the results of the model. To do this, we have to modify the `print.sienaBayes` function as NA's have crept into the output. In the function `print.sienaBayes` the parameters are rounded, which doesn't work if a vector has numbers and NA. This is not recognized as a *numeric* vector but as a XXX, which the `round` function doesn't accept. We will load the functions below, which will override the print function we loaded with the package. An alternative is to rename the function. 
```{r}
source("sienaprint_modified.R", echo=F)
```

```{r}
fit1
```

```{r}
sink("results_of_model.txt", append=T)
fit1
sink()
```

<!-- Instead of our model, we will be loading an *R Object* which contains fit1. Fit 1 is our simple model with all 18 teams on which I collected data. It took several minutes to be calculated. Enough to make a cup of coffee, check emails, keep on working on the script, switch computers and continue working on another task. CPU was close to 100 %, which is why universities should offer an **easy** way to analyze data on server space. Running `sienaBayes` gives you the total duration: Total duration 1481.995 seconds.   -->
<!-- ```{r} -->
<!-- load("fit1.RData") -->
<!-- ``` -->

#### Convergence Test

After you run a model, you should check convergence. The lines below will create a couple of plots. We are looking for any types of irregularities, upward, or downward trends. To run the convergence check we first need to load another R file with the necessary scripts. I did not want to see the output of running the file and hence I have added `echo=F` (You can write out `TRUE` or `FALSE` or add the shorthand `T` and `F`).
```{r}
source("BayesPlots.R", echo=F)
```

The following two lines of code will test convergence for every variable and for every team. If you want you can change folders before and after to save the files in a specific place or modify the function `RateTracePlots` and `NonRateTracePlots` to include an option for a file path. 
```{r}
RateTracePlots(fit1)
NonRateTracePlots(fit1)
```
In your working directory you should now see 10 pictures (png files). Instructions about what these are in the *BayesPlots.R* file we loaded. 

We are going to look at *fit1_NRTP_1.png*. First a bit of background. What did you do so far ? You run a number of [*Markov Chain Monte Carlo* (MCMC)](http://twiecki.github.io/blog/2015/11/10/mcmc-sampling/) simulations using Bayesian regression. The goal of this is to find the values for your variables. The provided link provides an excellent explanation of how this is done. In this example, you ran 35 simulations (`nwarm = 10` and `nmain = 25`). At the end of every simulation, your variables were given certain values. These values you see as dots on your graph. Convergence happens when the values are similar in each run of the simulation. With our current analysis, it is not possible to make any claims about that. Theta is to Bayesian calculation what beta is to classical frequentist calculations. 

<!-- ## ADD in folder: -->
<!-- example of good and bad convergence -->

#### Improve the model
As with any other model, you can now decide to add or remove effects until you answered all your questions. For example, we can test for balance (*transTrip*). We'll add the effects to our effect object *ir_effect*, and run the model using `sienaBayes`. We are using a new fit name `fit2` so that we look again at the results we want, extract numbers from `fit1`. If you are running into memory problems, then you might want to overwrite the model. You could first save `fit1` as an *Rdata* file (`save(object, file = filepath)`). We did a couple of changes to the sienaBayes call. We increased the length of the Markov Chain, by increasing `nwarm` and `nmain`, and asked for a `silentstart` to reduce the output we get in the console. Running this model takes 58 seconds.


<!-- # WHY CAN"T HE NOT CALCUALTE A STANDARD DEVIATION? -->
```{r}
ir_effect<-includeEffects(ir_effect, transTrip)
fit2 <- sienaBayes(ir_algo, data=ir, effects=ir_effect, nwarm=20, nmain=70, nrunMHBatches=10, silentstart=T)
```

```{r}
fit2
```

```{r}
sink("results_of_model.txt", append=T)
fit2
sink()
```

Again, we'll be inspecting the convergence plots.
```{r}
RateTracePlots(fit2)
NonRateTracePlots(fit2)
```

Wow, that backfired. There is a slight backward trend for information retrieval estimates for t1 in Group 1. The estimates for the dyadic and individual covariates are also fluctuating. If this would happen, you should first test the robustness of these results by modifying the parameters for the Markov Chain. For example, you could run: `fit3 <- sienaBayes(ir_algo, data=ir, effects=ir_effect, nwarm=20, nmain=100, nrunMHBatches=10, silentstart=T, initgainGlobal = 0, initgainGroupwise = 0)`.

To inspect the model, run convergence test you can load the saved model fit3.
```{r}
fit3 <- sienaBayes(ir_algo, data=ir, effects=ir_effect, nwarm=20, nmain=100, nrunMHBatches=10, silentstart=T, initgainGlobal = 0, initgainGroupwise = 0)
RateTracePlots(fit3)
NonRateTracePlots(fit3)
```
```{r}
fit3
```

If this is all the data that you have, I would suggest trying to add simple network effects, increase the Markov Chain, but to not have high hopes to ever get good results. Three teams with 5 people is a very small dataset.  

## Running a REM

### Background

#### Context
We analyzed information exchange in several emergency care teams. These teams were operating in the local simulation center and practicing a specific procedure (ABCDE). All training sessions were recorded and manually coded. A training session had most often 4 team members (main nurse, supporting nurse, doctor, and specialist). The specialist was only called in to transfer the simulated patient from the emergency care room to the care-giving unit. 
More information about the context is available in [The Main Nurse as a Linchpin in Emergency Care Teams](https://www.researchgate.net/project/May-I-ask-you-The-influence-of-individual-dyadic-and-network-factors-on-the-emergence-of-information-exchange-in-teams).

#### Variables
The variables we collected through coding the videos were *information retrieval* and *information allocation* (dependent variables), and three forms of higher order processing of information (*summarizing*, *elaboration*, and *decision-making*). Our coding schema also included exchanges between a human and a machine, when for example, a nurse is working on a machine. 
Additionally we collected the following variables using a survey: Awareness of team member's expertise (*knowing*; independent variable), the importance team member's attach to each other's expertise (*valuing*, independent variable), how adaptive individuals are (*adaptive expertise*; independent variable), their emotional attachment to their group (*social identity*; independent variable), and background variables (*gender*, *age*, *nationality*, *job role*, *tenure*, *number of training session*, *department*; control variable).

#### What we did
We analyzed the data using relational event modeling. For the chapter mentioned above we created some specific scripts to deal with self-loops, valued independent data etc. Below I created a simpler script to demonstrate how to run a REM> 

### REM Steps
For this workshop we will be focusing

In general, the steps are: 
1. Import the data
2. Specify independent, dependent variables
3. Select the effects and specify the model
4. Test goodness of fit

#### Running a REM

<!--We will begin by using a simple relational event analysis, using the relevent package. The advantage is that you don't have to do much coding with that. -->

```{r}
library(relevent)
library(informR)
source("rem_data_loading.R", echo=F)
```

`rem` can be applied to egocentric relational event data but requires that the user supplies the necessary statistics. On the other hand, `rem.dyad` is less flexible, but has more built-in functionalities (and hence requires less coding). The function takes the form of `rem.dyad(edgelist, n, effects = NULL, ordinal = TRUE`. To run it, you need to define an edgelist, the number of senders and receivers, an optional list of effects, and indicate if the timing is ordinal (TRUE) or if the exact timing of events should be used (`ordinal=FALSE`). The edgelist is a 3-column matrix which contains information about the timing, sender, and receiver. The data needs to be sorted by time. 

```{r}
nbr_send_rec = c(unique(remj[remj$Observation == 10,2]), unique(remj[remj$Observation == 10,(3)]))
nbr_send_rec
```


```{r}
fit1 <- rem.dyad(remj[remj$Observation == 10,(1:3)], n = length(nbr_send_rec), ordinal=FALSE) 
summary(fit1)
```
The model fit1 only includes the fixed effect. Not interesting at all. We are going to add a bunch of effects. 
```{r}
fit2 <- rem.dyad(remj[remj$Observation == 10,(1:3)], n = length(nbr_send_rec), ordinal=FALSE, effects= c("FESnd","FERec", "PSAB-BA", "PSAB-XA")) 
summary(fit2)
```

In my analysis I also wanted to take the type of event into account. Hence, I wanted to make a difference between information allocation,  information retrieval, summarizing, elaboration, and decision making. I also had individual variables I wanted to include. For these reasons I used `rem`.

The function `rem` requires a different set of arguments `rem(eventlist, statslist, supplist = NULL, timing = c("ordinal", "interval")`. The first, *eventlist* is a 2-column matrix (or list) with the time of an event and the event type. This is the file evlj. If you inspect its type (`class(evlj)`) and its structure (`str(evlj)`), you'll see that it is a list with 31 items. Each item in the list is a matrix with 2 columns. The first column in the matrix is a series of numbers, the event types, the second column contains the timing. 
```{r}
evlj$eventlist$`1`[1:10,]
evlj$event.key
```
This list was created by calling `gen.evl(eventlist, null.events=NULL)`. `Eventlist` is a 2 or 3 column matrix. Running `gen.evl` gives you an eventlist (a sequence of numbers indicating the events that occurred) and an event.key. 

The function `rem` also requires that you provide a *statslist*. This is a file containing some statistics about your data. The code below creates such as statslist for the fixed sender effects. Think of these as your intercepts only for sending information. 
```{r}
#Intercepts et cetera
evlj.ints <- gen.intercepts(evlj, contr = F)
nj<-7
njn<-c(1,2,3,4,5,6,10)
sformlistj <- c(
  lapply(2:(nj-1), function(z){
    str <- paste("^",njn[z], ".", sep="")
    grep(str, evlj$event.key[ ,2])
    # put in the square brackets, all events which are between real people.
    # the 2 stands for the event.key not event.id
  }),
  lapply(2:nj, function(z){
    str <- paste(".", njn[z],"$", sep = "")
   grep(str, evlj$event.key[, 2])
 })
)

b1j<-list()
b1.lj<-list()
FEsj<-list()

for(h in 1:length(evlj$eventlist)){
  b1j[[h]] <- lapply(sformlistj, function(x) evlj.ints[[h]][[1]][, , evlj$event.key[x,2]])
  b1.lj[[h]] <- lapply(b1j[[h]], apply, MARGIN = 1:2, sum)
  FEsj[[h]] <- array(unlist(b1.lj[[h]]), dim = c(nrow(b1.lj[[h]][[1]]), ncol(b1.lj[[h]][[1]]),
                                                 length(b1.lj[[h]])))
  dimnames(FEsj[[h]]) <- list(dimnames(b1j[[h]][[1]])[[1]], dimnames(b1j[[h]][[1]])[[2]],
                              c(paste("FESnd", njn[-c(1,7)], sep="."),paste("FERec", njn[-1], sep=".")))
}

#Simple Fixed Effects for Sender and Receiver
FEsj<-sfl2statslist(FEsj)
```

Now we have the statslist. The *supplist* has been created when loading the data. The goal of the *supplist* is to indicate which events could have been observed and which not. For example, in my setting the doctor entered the emergency care room at a later stage. This means that it was impossible to observe any interaction with the doctor before that. My event list contains an event 'doctor enters the room' (aka handover between nurse and doctor). We used this event to indicate when interaction with the doctor was possible. 

fit1.rem is running a simple fixed effect model. This is the null model only including intercepts.
```{r}
#Model 1 Fixed Effects for Sender and Receiver (pooled likelihood)
fit1.rem<-rem(evlj$eventlist,FEsj,timing="interval",estimator="MLE",supplist=supplistj)
summary(fit1.rem)
```

Now we will be including a participation shift event. 10 is a special 'ego actor': The team. When team members engaged in exchanges such as summarizing information, elaboration of information, and decision-making it was difficult to decide who the receiver of this information exchange is. Therefore, we coded them all to be directed at the team (actor 10). For this reason, actor 10 cannot send out information exchanges. 
```{r}
#Ignore null events and events involving 10, who cannot by construct reply.
nevs<-grep(paste(c(paste("(",evlj$null.events,")",sep=""),"(10)"),collapse="|"),evlj$event.key[,2])
evk.ex<-cbind(evlj$event.key[-nevs,1],do.call("rbind",strsplit(evlj$event.key[-nevs,2],"\\.")))

#Match AB->BA turn taking PShifts
tr1<-cbind(evk.ex[,1],evk.ex[sapply(paste(evk.ex[,2],evk.ex[,3],sep="."),function(x) which(x==paste(evk.ex[,3],evk.ex[,2],sep="."))),1])
tr2<-paste(tr1[,1],tr1[,2],sep="")

tr.sfl<-glb.sformlist(evlj,sforms=list(tr2),new.names="PS-ABBA",interval=TRUE,cond=FALSE)

FEsTr<-slbind(tr.sfl,FEsj)
#FEsTr<-slbind(tr.sfl,KWSnd1) 
fit2.rem <- rem(evlj$eventlist, FEsTr, supplist=supplistj,estimator = "MLE",timing="interval")

summary(fit2.rem)
```
A quick note about the interpretation of the results. The estimates are hazard rates. You could take the exponential of them (`exp(-2.85)`) to get log values. In that way, you can say thinks like "the likelihood that the doctor (actor 3) will send information is 95 per cent  (exp(-2.85) less likely than that the main nurse will send information". Why the main nurse? You have to specify a base level. We picked the main nurse as this actor initiates the interaction. However, we could have also picked someone else. 
The danger with using log likelihoods when discussing your results is that others might not view it as a longitudinal study. An alternative is to talk about *hazard rates*. You would then say something like "The rate at which the doctor sends information is 95 % less fast than the rate of sending information from the main nurse."
