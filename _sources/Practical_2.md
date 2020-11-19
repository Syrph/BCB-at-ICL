---
title: ""
permalink: /Practical_2/
excerpt: ""
last_modified_at: 2020-07-27
redirect_from:
  - /theme-setup/
layout: single
classes: wide
sidebar:
  nav: docs
---

## Phylogenetics in R

### 1. Introduction and resources

This practical introduces you to basic phylogenetic computing in `R`. We will review importing phylogenetic trees as data files, displaying phylogenetic trees visually, and some basic evolutionary computations that can be conducted with phylogenetic trees. This practical will deliver some of the important background for Coursework 1. Below you will find some of the relevant resources required for this practical.

Parts (sections 2,3,4) of this practical are written by [Natalie Cooper](http://nhcooper123.github.io/).
The original can be found [here](https://github.com/nhcooper123/TeachingMaterials/blob/master/PhD_Museum/VisualisingPhylo.Rmd). Whereas, parts (section 5 & 6) of this practical are written by Adam Devenish & Rob Barber (a.devenish@imperial.ac.uk) (r.barber19@imperial.ac.uk).

#### Installing and loading extra packages in R
To plot phylogenies (or use any specialized analysis) in R, you need to download one or more additional packages from the basic R installation. For this practical you will need to load the following packages:

• ape

• phytools

We've installed these packages in the last practical by using the code `source("install.R")`, but they don't automatically get loaded into your `R` session. 
Instead you need to tell `R` to load them **every time** you start a new `R` session and want to use functions from these packages. 
To load the package `ape` into your current `R` session:


```R
library(ape)
```

You can think of `install.packages` like installing an app from the App Store on your smart phone - you only do this once - and `library` as being like pushing the app button on your phone - you do this every time you want to use the app.

Don't forget to load `phytools` too!


```R
library(phytools)
```

    Loading required package: maps
    


### 2. A refresher of phylogenetic trees

This section will review some basic aspects of phylogenetic trees and introduce how trees are handled at the level of software. Because you are now interacting with phylogenetic trees at a ‘lower level’ (i.e. at the bioinformatics level), it is also helpful to know some of the names for parts of phylogenetic trees used in computer science.

#### A. Tree parameters
 
A phylogenetic tree is an ordered, multifurcating graph with labeled **tips** (or **leaves**) (and sometimes labeled histories). It represents the relative degrees of relationships of species (i.e. tips or OTUs). The graph consists of a series of **branches** (or **edges**) with join successively towards **nodes** (or **vertices**, *sing.* **vertex**). Each node is subtended by a single branch, representing the lineage of ancestors leading to a node. The node is thus the common ancestor of two or more descendant branches. All the descendant branches of a given node (and all of the their respective descendants) are said to form a **clade** (or **monophyletic group**).


```R
# rtree creates a random tree. Setting the seed means that the same 'random' tree will plotted each time. This is because random
# numbers from your computer are generated using the internal clock. Google it for more info!
set.seed(0); plot(rtree(10), "unrooted")

# Plot clade label
rect(1.3,2.0,2.5,2.7, border = "grey")
text(2.8, 2.5, "Clade")

# Plot node label
arrows(0.35,0.9,0.75,1, length = 0.125, angle = 20, code = 1)
text(1.0, 1, "Node")

# Plot edge label
arrows(0.65,1.45,0.15,1.4, length = 0.125, angle = 20, code = 1)
text(-0.1, 1.4, "Edge")

# Plot tip label
arrows(-0.15,2.8,-0.45,2.95, length = 0.125, angle = 20, code = 1)
text(-0.6, 3, "Tip")
```


```{image} Practical_2_9_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```


When we select a node to act as the base of a tree, the tree is said to be **rooted**. At the bottom of a tree, is the **root node** (or simply the **root**).


```R
# You can also read in trees directly from text by specifying the branch lengths after each node/tip. More info below!
tree <- read.tree(text = "(((Homo:1, Pan:1):1, Gorilla:1):1, Pongo:1);")
plot(tree)

# Plot root label
lines(c(-0.5,0), c(3.18,3.18))
arrows(0.03,3.18,0.35,3.18, length = 0.125, angle = 20, code = 1)
text(0.5, 3.18, "Root")
```


```{image} Practical_2_11_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



Phylogenetic trees of the kind shown above are fairly simple and lack information about time or character changes occurring along a branch. We can assign branch length in the form of either time or the amount of change/substitution along a branch. A tree with **branch lengths** depicted can be called a **phylogram**.

When (an implied) dimension of time is being considered, all the tips of the tree must be at the level representing the time in which they are observed. For trees where all the species are extant, the tips are flush at the top. This representation is called an **ultrametric** tree.


```R
tree <- read.tree(text = "(((Homo:6.3, Pan:6.3):2.5, Gorilla:8.8):6.9, Pongo:15.7);")
plot(tree)
axisPhylo()
```


```{image} Practical_2_13_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```


#### B. Informatic representations of tree

To perform any useful calculations on a tree, we need both a computer-readable tree format and (in part) to understand how trees are constructed in computer memory.
 
#### Text based formats

Storage of trees for transfer between different software is essential. This is most commonly achieved with a text-based format stored in a file. The most common file format for representing phylogenetic trees is **Newick format**. This consists of clades represented within parentheses. Commas separate each clade. Either tip names or symbols representing the tips are nested within the lowest orders of parentheses. Each tip or branch can be associated with a branch length scalar that follows a colon.

For example:

`"(((Homo, Pan), Gorilla), Pongo);"`

Or with branch length:

`"(((Homo:6.3, Pan:6.3):2.5, Gorilla:8.8):6.9, Pongo:15.7);"`

Trees are also increasing use of XML formats such as PhyloXML and NeXML.

In this practical we are going to use the `elopomorph.tre` newick tree.
You can open it with a simple text editor to see the newick tree structure.

#### Edge table

It is also possible to represent a phylogenetic tree as a matrix of edges and vertices called an edge table. This is an even less intuitive representation, but it is implemented in `R` and worth reviewing here.

There are a number of conventions that can be used to create an edge table. The general concept consists of numbering the tips *1 - n*, and all internal nodes labeled *n+1 ... n+n-1*. The numbers for the internal nodes can be assigned arbitrarily or according to an algorithm.

In `R` packages like `ape`, edge tables are constructed as follows:

| node | connects to |
|---|---|
| 5 | 6 |
| 6 | 7 |
| 7 | 1 |
| 7 | 2 |
| 6 | 3 |
| 5 | 4 |

You read the table as follows: node `5` (root) connects to node `6`. The node `6` connects to node `7`. Node `7` connects to node `1` that happen to be the first tip (`Homo`) and to node `2` (`Pan`) etc... Note that in a binary tree (i.e. a tree where each node has only two descendants) each node always connects to two elements (nodes or tips).


```R
tree <- read.tree(text = "(((Homo, Pan), Gorilla), Pongo);")
plot(tree, label.offset = 0.1)
nodelabels() ; tiplabels()
```


```{image} Practical_2_15_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



#### Records & pointers

At a lower level, phylogenetic trees can be represented in computer memory as more complex data objects. We don’t need to go into detail here, but if you consider nodes and tips as data objects (i.e. a dataframe), a tree could be stored as an array of dataframes which store information about which store information about which members of that same array are descendants and which are ancestors.

### 3. Loading your phylogeny and data into `R`
#### Reading in a phylogeny from a file
To load a tree you need the function `read.tree`.
`read.tree` can read any newick format trees (see above) like the `elopomorph.tre` file.


```R
fishtree <- read.tree("elopomorph.tre")
```

**Be sure you are always in the right directory. Remember you can navigate in `R` using `setwd()`, `getwd()` and `list.files()` (to see what's in the current directory). While using this notebook all the files are stored in the main binder, so you shouldn't need to change the directory.**

#### Reading in a phylogeny that is already built into `R`
The bird and anole phylogenies are already built into `R` so we don't need to read them in using `read.tree`.
Instead we just use:



```R
data(bird.orders)
data(anoletree)
```

#### Reading and viewing your data in `R`
Later we will use some Greater Antillean *Anolis* lizard data to add data to a phylogeny.
Before we can add data to our tree, we need to load the data we are going to use. 
`R` can read files in lots of formats, including comma-delimited and tab-delimited files.
Excel (and many other applications) can output files in this format (it's an option in the `Save As` dialogue box under the `File` menu). 
To save time I have given you a comma-delimited text file called `anole.data.csv` which we are going to use. 
Load these data as follows. 
I am assuming you have set your working directory, if not don't forget the path.


```R
anoledata <- read.csv("anole.data.csv", header = TRUE)
```

You can use `read.delim` for tab delimited files or `read.csv` for comma delimited files (**c**omma **s**eparated **v**alues).
`header = TRUE`, indicates that the first line of the data contains column headings.

This is a good point to note that unless you **tell** `R` you want to do something, it won't do it automatically. 
So here if you successfully entered the data, `R` won't give you any indication that it worked.
Instead you need to specifically ask `R` to look at the data.

We can look at the data by typing:


```R
str(anoledata)
```

    'data.frame':	100 obs. of  23 variables:
     $ species               : chr  "ahli" "alayoni" "alfaroi" "aliniger" ...
     $ AVG.SVL               : num  4.04 3.82 3.53 4.04 4.38 ...
     $ AVG.hl                : num  2.88 2.7 2.38 2.9 3.36 ...
     $ AVG.hw                : num  2.36 1.99 1.56 2.37 2.69 ...
     $ AVG.hh                : num  2.13 1.75 1.39 2.05 2.32 ...
     $ AVG.ljl               : num  2.85 2.71 2.32 2.9 3.38 ...
     $ AVG.outlever          : num  2.75 2.62 2.26 2.83 3.29 ...
     $ AVG.jugal.to.symphysis: num  2.54 2.37 2.08 2.6 3.07 ...
     $ AVG.femur             : num  2.74 2.07 2.17 2.48 2.8 ...
     $ AVG.tibia             : num  2.69 2.02 2.09 2.34 2.69 ...
     $ AVG.met               : num  2.25 1.54 1.55 1.87 2.18 ...
     $ AVG.ltoe.IV           : num  2.55 1.88 1.73 2.26 2.53 ...
     $ AVG.toe.IV.lam.width  : num  0.1795 0.0488 -0.5361 0.4904 0.8441 ...
     $ AVG.humerus           : num  2.46 1.95 1.63 2.3 2.62 ...
     $ AVG.radius            : num  2.27 1.69 1.4 2.09 2.34 ...
     $ AVG.lfing.IV          : num  1.94 1.4 1.04 1.7 1.98 ...
     $ AVG.fing.IV.lam.width : num  0.0754 -0.0739 -0.755 0.3155 0.6584 ...
     $ AVG.pelv.ht           : num  1.99 1.51 1.19 1.87 2.1 ...
     $ AVG.pelv.wd           : num  1.71 1.419 0.946 1.752 2.014 ...
     $ Foot.Lam.num          : num  3.28 3.43 3.2 3.58 3.72 ...
     $ Hand.Lam.num          : num  2.87 3.08 2.73 3.16 3.24 ...
     $ Avg.lnSVL2            : num  3.94 3.74 3.48 3.93 4.36 ...
     $ Avg.ln.t1             : num  4.41 4 4.37 4.44 5.04 ...


**Always** look at your data before beginning any analysis to check it read in correctly.

`str` shows the structure of the data frame (this can be a really useful command when you have a big data file). 
It also tells you what kind of variables `R` thinks you have (characters, integers, numeric, factors etc.). 
Some `R` functions need the data to be certain kinds of variables so it's useful to check this.


```R
head(anoledata)
```


<table>
<caption>A data.frame: 6 × 23</caption>
<thead>
	<tr><th></th><th scope=col>species</th><th scope=col>AVG.SVL</th><th scope=col>AVG.hl</th><th scope=col>AVG.hw</th><th scope=col>AVG.hh</th><th scope=col>AVG.ljl</th><th scope=col>AVG.outlever</th><th scope=col>AVG.jugal.to.symphysis</th><th scope=col>AVG.femur</th><th scope=col>AVG.tibia</th><th scope=col>⋯</th><th scope=col>AVG.humerus</th><th scope=col>AVG.radius</th><th scope=col>AVG.lfing.IV</th><th scope=col>AVG.fing.IV.lam.width</th><th scope=col>AVG.pelv.ht</th><th scope=col>AVG.pelv.wd</th><th scope=col>Foot.Lam.num</th><th scope=col>Hand.Lam.num</th><th scope=col>Avg.lnSVL2</th><th scope=col>Avg.ln.t1</th></tr>
	<tr><th></th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>⋯</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>ahli    </td><td>4.039125</td><td>2.882657</td><td>2.356126</td><td>2.131599</td><td>2.854745</td><td>2.754934</td><td>2.538052</td><td>2.741378</td><td>2.686032</td><td>⋯</td><td>2.460728</td><td>2.268511</td><td>1.943288</td><td> 0.07541664</td><td>1.987646</td><td>1.7095849</td><td>3.279600</td><td>2.866197</td><td>3.939015</td><td>4.406652</td></tr>
	<tr><th scope=row>2</th><td>alayoni </td><td>3.815705</td><td>2.702116</td><td>1.990610</td><td>1.751588</td><td>2.708966</td><td>2.617852</td><td>2.371995</td><td>2.065121</td><td>2.016402</td><td>⋯</td><td>1.947873</td><td>1.687093</td><td>1.403029</td><td>-0.07391568</td><td>1.507128</td><td>1.4194873</td><td>3.432372</td><td>3.075269</td><td>3.743863</td><td>4.002788</td></tr>
	<tr><th scope=row>3</th><td>alfaroi </td><td>3.526655</td><td>2.378156</td><td>1.556037</td><td>1.390037</td><td>2.323857</td><td>2.263844</td><td>2.077565</td><td>2.172476</td><td>2.094946</td><td>⋯</td><td>1.628260</td><td>1.401183</td><td>1.040277</td><td>-0.75502258</td><td>1.189367</td><td>0.9458495</td><td>3.198016</td><td>2.733866</td><td>3.478776</td><td>4.369448</td></tr>
	<tr><th scope=row>4</th><td>aliniger</td><td>4.036557</td><td>2.898836</td><td>2.366592</td><td>2.046660</td><td>2.903946</td><td>2.828555</td><td>2.596821</td><td>2.475445</td><td>2.344015</td><td>⋯</td><td>2.298878</td><td>2.091864</td><td>1.702199</td><td> 0.31554040</td><td>1.867949</td><td>1.7521520</td><td>3.582875</td><td>3.156774</td><td>3.933181</td><td>4.441203</td></tr>
	<tr><th scope=row>5</th><td>allisoni</td><td>4.375390</td><td>3.358957</td><td>2.690339</td><td>2.317309</td><td>3.381306</td><td>3.288402</td><td>3.071149</td><td>2.795145</td><td>2.687734</td><td>⋯</td><td>2.618551</td><td>2.341565</td><td>1.983412</td><td> 0.65838319</td><td>2.097302</td><td>2.0142361</td><td>3.721185</td><td>3.239211</td><td>4.355582</td><td>5.039851</td></tr>
	<tr><th scope=row>6</th><td>allogus </td><td>4.040138</td><td>2.861027</td><td>2.351275</td><td>2.142107</td><td>2.848331</td><td>2.750404</td><td>2.539320</td><td>2.739175</td><td>2.684476</td><td>⋯</td><td>2.459994</td><td>2.267176</td><td>1.919010</td><td> 0.05329130</td><td>2.008426</td><td>1.6875679</td><td>3.335990</td><td>2.808270</td><td>4.028863</td><td>4.510920</td></tr>
</tbody>
</table>



This gives you the first few rows of data along with the column headings.


```R
names(anoledata)
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class="list-inline"><li>'species'</li><li>'AVG.SVL'</li><li>'AVG.hl'</li><li>'AVG.hw'</li><li>'AVG.hh'</li><li>'AVG.ljl'</li><li>'AVG.outlever'</li><li>'AVG.jugal.to.symphysis'</li><li>'AVG.femur'</li><li>'AVG.tibia'</li><li>'AVG.met'</li><li>'AVG.ltoe.IV'</li><li>'AVG.toe.IV.lam.width'</li><li>'AVG.humerus'</li><li>'AVG.radius'</li><li>'AVG.lfing.IV'</li><li>'AVG.fing.IV.lam.width'</li><li>'AVG.pelv.ht'</li><li>'AVG.pelv.wd'</li><li>'Foot.Lam.num'</li><li>'Hand.Lam.num'</li><li>'Avg.lnSVL2'</li><li>'Avg.ln.t1'</li></ol>

This gives you the names of the columns.

### 4. Basic tree viewing in `R`
Now let's visualise some phylogenies! We'll use the Elopomorpha (eels and similar fishes) tree to start as it is simple.


```R
fishtree <- read.tree("elopomorph.tre")
```

Let's examine the tree by typing:


```R
fishtree
str(fishtree)
```


    
    Phylogenetic tree with 62 tips and 61 internal nodes.
    
    Tip labels:
      Moringua_edwardsi, Kaupichthys_nuchalis, Gorgasia_taiwanensis, Heteroconger_hassi, Venefica_proboscidea, Anguilla_rostrata, ...
    
    Rooted; includes branch lengths.


    List of 4
     $ edge       : int [1:122, 1:2] 63 64 64 65 66 67 68 68 69 70 ...
     $ edge.length: num [1:122] 0.0105 0.0672 0.00537 0.00789 0.00157 ...
     $ Nnode      : int 61
     $ tip.label  : chr [1:62] "Moringua_edwardsi" "Kaupichthys_nuchalis" "Gorgasia_taiwanensis" "Heteroconger_hassi" ...
     - attr(*, "class")= chr "phylo"
     - attr(*, "order")= chr "cladewise"


`fishtree` is a fully resolved tree with branch lengths. 
There are 62 species and 61 internal nodes. 
We can plot the tree by using the `plot.phylo` function of `ape`. 
Note that we can just use the function `plot` to do this as `R` knows if we ask it to plot a phylogeny to use `plot.phylo` instead!


```R
plot(fishtree, cex = 0.5)
```


```{image} Practical_2_37_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



`cex = 0.5` reduces the size of the tip labels so we can read them. 

We can also zoom into different sections of the tree that you're interested in:


```R
zoom(fishtree, grep("Gymnothorax", fishtree$tip.label), subtree = FALSE, cex = 0.8)
```


```{image} Practical_2_39_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



The `grep` function is a generic function in `R` that allows to *grab* any element in an object containing the desired characters.
In this example, `grep` is going to search for all the elements in `fishtree$tip.label` that contains `Gymnothorax` (e.g. `Gymnothorax_kidako`, `Gymnothorax_reticularis`).
Try using only `grep("thorax", fishtree$tip.label)` to see if it also only selects the members of the *Gymnothorax* genus.

In this example, we just display the tree for the *Gymnothorax* genus but you can also see how the species fit into the rest of the tree using:



```R
zoom(fishtree, grep("Gymnothorax", fishtree$tip.label), subtree = TRUE, cex = 0.8)
```


```{image} Practical_2_41_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



Note that `zoom` is a specific plotting function that will automatically set the plotting window to display two plots at once. This might create some conflicts if you're using RStudio. The bug can be easily solved though by typing `dev.off()` to reinitialise the plotting window and then proceed to the normal `zoom(...)` function as written above.

You can also reset this to one plot only per window by using:



```R
par(mfrow = c(1, 1))
```

To get further options for the plotting of phylogenies:


```R
?plot.phylo
```

Using the question mark (`?`) can also be done for every function if you want more details!

Note that although you can use `plot` to plot the phylogeny, you need to specify `plot.phylo` to find out the options for plotting trees. You can change the style of the tree (`type`), the color of the branches and tips (`edge.color`, `tip.color`), and the size of the tip labels (`cex`). 
Here's an fun/hideous example! 



```R
plot(fishtree, type = "unrooted", edge.color = "deeppink", tip.color = "springgreen",  cex = 0.5)
```


```{image} Practical_2_47_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



Or try


```R
plot(ladderize(fishtree), type = "c", edge.color = "darkviolet", tip.color = "hotpink",  cex = 0.5)
```


```{image} Practical_2_49_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



The `ladderize` function allows to display the branches from shortest to longest.

> Try to modify the graphical options (colors, display, size, ordering of the nodes, etc.) to obtain the most beautiful or ugliest Elopomorpha phylogeny!

### 5. Manipulating phylogenetic trees in `R`

There are a range of ways in which we can manipulate trees in R. To start lets take a look at the bird family Turdidae.


```R
Turdidae_tree <- read.nexus("Turdidae_birdtree.nex")
```

As this multiphylo object (i.e. contains 100 different trees) we need to first choose one random tree before we start.


```R
Ran_Turdidae_tree <- sample(Turdidae_tree,size=1)[[1]]
plotTree(Ran_Turdidae_tree,type="fan",fsize=0.4,lwd=0.5,ftype="i")
```


```{image} Practical_2_55_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



First, lets see what species are in the tree.


```R
Ran_Turdidae_tree$tip.label
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class="list-inline"><li>'Zoothera_everetti'</li><li>'Zoothera_naevia'</li><li>'Zoothera_pinicola'</li><li>'Hylocichla_mustelina'</li><li>'Catharus_aurantiirostris'</li><li>'Catharus_mexicanus'</li><li>'Catharus_dryas'</li><li>'Catharus_fuscater'</li><li>'Catharus_ustulatus'</li><li>'Catharus_bicknelli'</li><li>'Catharus_fuscescens'</li><li>'Catharus_minimus'</li><li>'Catharus_frantzii'</li><li>'Catharus_guttatus'</li><li>'Catharus_occidentalis'</li><li>'Catharus_gracilirostris'</li><li>'Entomodestes_coracinus'</li><li>'Entomodestes_leucotis'</li><li>'Cichlopsis_leucogenys'</li><li>'Turdus_mupinensis'</li><li>'Psophocichla_litsitsirupa'</li><li>'Turdus_viscivorus'</li><li>'Turdus_menachensis'</li><li>'Turdus_plebejus'</li><li>'Turdus_helleri'</li><li>'Turdus_olivaceus'</li><li>'Turdus_ludoviciae'</li><li>'Turdus_iliacus'</li><li>'Turdus_lherminieri'</li><li>'Turdus_subalaris'</li><li>'Turdus_tephronotus'</li><li>'Turdus_merula'</li><li>'Turdus_pelios'</li><li>'Turdus_leucops'</li><li>'Turdus_fulviventris'</li><li>'Turdus_olivater'</li><li>'Turdus_serranus'</li><li>'Turdus_fuscater'</li><li>'Turdus_nigriceps'</li><li>'Turdus_chiguanco'</li><li>'Nesocichla_eremita'</li><li>'Turdus_ignobilis'</li><li>'Turdus_maranonicus'</li><li>'Turdus_amaurochalinus'</li><li>'Turdus_reevei'</li><li>'Turdus_rufopalliatus'</li><li>'Turdus_lawrencii'</li><li>'Turdus_flavipes'</li><li>'Turdus_obsoletus'</li><li>'Turdus_albicollis'</li><li>'Turdus_assimilis'</li><li>'Turdus_fumigatus'</li><li>'Turdus_hauxwelli'</li><li>'Turdus_leucomelas'</li><li>'Turdus_rufiventris'</li><li>'Turdus_grayi'</li><li>'Turdus_haplochrous'</li><li>'Turdus_nudigenis'</li><li>'Turdus_maculirostris'</li><li>'Turdus_daguae'</li><li>'Turdus_falcklandii'</li><li>'Turdus_olivaceofuscus'</li><li>'Turdus_libonyanus'</li><li>'Turdus_bewsheri'</li><li>'Turdus_rufitorques'</li><li>'Turdus_migratorius'</li><li>'Turdus_infuscatus'</li><li>'Turdus_nigrescens'</li><li>'Turdus_plumbeus'</li><li>'Turdus_aurantius'</li><li>'Turdus_jamaicensis'</li><li>'Turdus_swalesi'</li><li>'Turdus_boulboul'</li><li>'Turdus_hortulorum'</li><li>'Turdus_cardis'</li><li>'Turdus_unicolor'</li><li>'Turdus_dissimilis'</li><li>'Turdus_rubrocanus'</li><li>'Turdus_albocinctus'</li><li>'Turdus_kessleri'</li><li>'Turdus_pilaris'</li><li>'Turdus_ruficollis'</li><li>'Turdus_naumanni'</li><li>'Turdus_torquatus'</li><li>'Turdus_poliocephalus'</li><li>'Turdus_feae'</li><li>'Turdus_obscurus'</li><li>'Turdus_pallidus'</li><li>'Turdus_celaenops'</li><li>'Turdus_chrysolaus'</li><li>'Turdus_philomelos'</li><li>'Zoothera_guttata'</li><li>'Zoothera_gurneyi'</li><li>'Zoothera_wardii'</li><li>'Zoothera_crossleyi'</li><li>'Zoothera_oberlaenderi'</li><li>'Zoothera_machiki'</li><li>'Zoothera_piaggiae'</li><li>'Zoothera_tanganjicae'</li><li>'Zoothera_major'</li><li>'Zoothera_princei'</li><li>'Cataponera_turdoides'</li><li>'Zoothera_camaronensis'</li><li>'Zoothera_imbricata'</li><li>'Zoothera_spiloptera'</li><li>'Zoothera_citrina'</li><li>'Zoothera_interpres'</li><li>'Zoothera_dohertyi'</li><li>'Zoothera_leucolaema'</li><li>'Zoothera_peronii'</li><li>'Zoothera_cinerea'</li><li>'Zoothera_mendeni'</li><li>'Zoothera_erythronota'</li><li>'Zoothera_turipavae'</li><li>'Zoothera_schistacea'</li><li>'Zoothera_margaretae'</li><li>'Zoothera_sibirica'</li><li>'Zoothera_lunulata'</li><li>'Zoothera_heinei'</li><li>'Zoothera_dumasi'</li><li>'Zoothera_talaseae'</li><li>'Zoothera_dauma'</li><li>'Zoothera_andromedae'</li><li>'Zoothera_monticola'</li><li>'Zoothera_joiceyi'</li><li>'Zoothera_marginata'</li><li>'Zoothera_mollissima'</li><li>'Zoothera_dixoni'</li><li>'Cochoa_purpurea'</li><li>'Cochoa_azurea'</li><li>'Cochoa_viridis'</li><li>'Cochoa_beccarii'</li><li>'Chlamydochaera_jefferyi'</li><li>'Geomalia_heinrichi'</li><li>'Sialia_currucoides'</li><li>'Sialia_mexicana'</li><li>'Sialia_sialis'</li><li>'Myadestes_genibarbis'</li><li>'Myadestes_ralloides'</li><li>'Myadestes_melanops'</li><li>'Myadestes_coloratus'</li><li>'Myadestes_palmeri'</li><li>'Myadestes_elisabeth'</li><li>'Myadestes_townsendi'</li><li>'Myadestes_obscurus'</li><li>'Myadestes_occidentalis'</li><li>'Myadestes_unicolor'</li><li>'Myadestes_lanaiensis'</li><li>'Neocossyphus_poensis'</li><li>'Stizorhina_fraseri'</li><li>'Stizorhina_finschi'</li><li>'Neocossyphus_rufus'</li><li>'Alethe_fuelleborni'</li><li>'Alethe_choloensis'</li><li>'Alethe_poliophrys'</li><li>'Alethe_poliocephala'</li><li>'Alethe_diademata'</li><li>'Myophonus_borneensis'</li><li>'Myophonus_castaneus'</li><li>'Myophonus_melanurus'</li><li>'Myophonus_blighi'</li><li>'Myophonus_robinsoni'</li><li>'Myophonus_insularis'</li><li>'Myophonus_horsfieldii'</li><li>'Myophonus_caeruleus'</li><li>'Myophonus_glaucinus'</li><li>'Brachypteryx_stellata'</li><li>'Brachypteryx_montana'</li><li>'Brachypteryx_hyperythra'</li><li>'Brachypteryx_leucophrys'</li><li>'Heinrichia_calligyna'</li></ol>



Lets say we want to drop all species with the Myadestes genus. In this instance we first find all the associated tip.labels. 


```R
drop.species <- ("Myadestes")

# sapply is a function that will iterate a given function over a vector (check out apply, lapply, mapply for more info)
# in this case, we're using the grep function to find the index of anything in tip.labels that matches drop.species
selected.tips <- sapply(drop.species,grep,Ran_Turdidae_tree$tip.label)

# We then use the indices to select only the tips we want from all the available tip labels
drop.species <- Ran_Turdidae_tree$tip.label[selected.tips]
drop.species
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class="list-inline"><li>'Myadestes_genibarbis'</li><li>'Myadestes_ralloides'</li><li>'Myadestes_melanops'</li><li>'Myadestes_coloratus'</li><li>'Myadestes_palmeri'</li><li>'Myadestes_elisabeth'</li><li>'Myadestes_townsendi'</li><li>'Myadestes_obscurus'</li><li>'Myadestes_occidentalis'</li><li>'Myadestes_unicolor'</li><li>'Myadestes_lanaiensis'</li></ol>



Now, we create a new tree, with the Myadestes tips dropped from it.


```R
# drop.tip will remove any matching tips from the tree
Ran_Turdidae_tree_NM<-drop.tip(Ran_Turdidae_tree,drop.species)
plotTree(Ran_Turdidae_tree_NM,type="fan",fsize=0.4,lwd=0.5,ftype="i")
```


```{image} Practical_2_61_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



Alternatively, lets say we want to extract the clade within the tree that includes the pre identified selected range of species.


```R
# Set diff will find all the tips that don't match drop.species, and then drop.tip will remove them
pruned_birdtree<-drop.tip(Ran_Turdidae_tree,
    setdiff(Ran_Turdidae_tree$tip.label,drop.species))
plotTree(pruned_birdtree,ftype="i")
```


```{image} Practical_2_63_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



**Remember, always to save your new revised tree. As this will save you time from having to prune it everytime. Use `write.tree` or `write.nexus` the same as you would to save a dataframe.**

For some analyses, you might want to work with on a genus level tree. This can easily be done by a few key steps. We can do this using base `R`, but we'll use some packages from the `tidyverse` that have some very useful functions. 
First load `stringr` (for manipulating strings) and `dplyr` (for manipulating dataframes). You'll see that it masks functions from other packages like stats. If you need those functions you can use code like this: `stats::filter()` to specify the package you want. 


```R
library(stringr)
library(dplyr)
```


```R
# Copy a list of all the tips from the tree
bird_tips <- Ran_Turdidae_tree$tip.label

# Split the labels into two strings where there's an underscore (simplfy returns the splits as separate columns in a dataframe)
bird_genera <- bird_tips %>% str_split(pattern = "_", simplify= TRUE)
colnames(bird_genera) <- c("Genus", "Species")

# Pull out the rows that have an distinct genus name. This will be the first instance in the dataframe for that genus.
bird_genera <- as.data.frame(bird_genera) %>% distinct(Genus, .keep_all = T)
bird_genera
```


<table>
<caption>A data.frame: 20 × 2</caption>
<thead>
	<tr><th scope=col>Genus</th><th scope=col>Species</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Zoothera      </td><td>everetti       </td></tr>
	<tr><td>Hylocichla    </td><td>mustelina      </td></tr>
	<tr><td>Catharus      </td><td>aurantiirostris</td></tr>
	<tr><td>Entomodestes  </td><td>coracinus      </td></tr>
	<tr><td>Cichlopsis    </td><td>leucogenys     </td></tr>
	<tr><td>Turdus        </td><td>mupinensis     </td></tr>
	<tr><td>Psophocichla  </td><td>litsitsirupa   </td></tr>
	<tr><td>Nesocichla    </td><td>eremita        </td></tr>
	<tr><td>Cataponera    </td><td>turdoides      </td></tr>
	<tr><td>Cochoa        </td><td>purpurea       </td></tr>
	<tr><td>Chlamydochaera</td><td>jefferyi       </td></tr>
	<tr><td>Geomalia      </td><td>heinrichi      </td></tr>
	<tr><td>Sialia        </td><td>currucoides    </td></tr>
	<tr><td>Myadestes     </td><td>genibarbis     </td></tr>
	<tr><td>Neocossyphus  </td><td>poensis        </td></tr>
	<tr><td>Stizorhina    </td><td>fraseri        </td></tr>
	<tr><td>Alethe        </td><td>fuelleborni    </td></tr>
	<tr><td>Myophonus     </td><td>borneensis     </td></tr>
	<tr><td>Brachypteryx  </td><td>stellata       </td></tr>
	<tr><td>Heinrichia    </td><td>calligyna      </td></tr>
</tbody>
</table>



We now have a dataframe of species, with one tip per each unique genus. 

A few things to note about the 'tidy' code we just ran:

`%>%` is an operator used to 'pipe' an object into a function. Piping is common in most computing languages so look it up for more information! It can make code less clutered by separating the data from the function you need to use. This isn't unique to `tidyverse` functions, but you'll see it used a lot more in their documentation.

For the `distinct` function, we specified a column name without using quotation marks. To make code easier to read, most tidy functions will process column names (or other labels) this way.

We can now use our unique species from each genera to drop all the other tips in the tree.


```R
# Combine the columns and add back in the underscore so they match the labels in the tree
genera_tips <- paste(bird_genera$Genus, bird_genera$Species, sep="_")

# Pull out the tips we want to drop
drop.tips <- setdiff(Ran_Turdidae_tree$tip.label, genera_tips)

# Remove all the species except one per genus
genera_tree<-drop.tip(Ran_Turdidae_tree, drop.tips)
plotTree(genera_tree,ftype="i")
```


![png](_images/Pratical_2_68_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



As the tree has dropped all but one species per genus, this means we will finally need to also rename the tip labels as well to reflect this change.



```R
# It's definitely worth checking that the labels match up properly when you change tip labels
par(mfrow=c(1,2))
plotTree(genera_tree,ftype="i")

# Swap species names for genera
genera_tree$tip.label<- bird_genera$Genus
plotTree(genera_tree,ftype="i")
```


```{image} Practical_2_70_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



It is important to note, that a big limitation with this approach is that by selecting only one species per genus to keep, that you may run the risk of unintentially dropping tips of species that are paraphyletic. For example, Zoothera genus is spread throughout Turdidae tree.


```R
plotTree(Ran_Turdidae_tree,fsize=0.2,lwd=0.2,ftype="i")
zoom(Ran_Turdidae_tree, grep("Zoothera", Ran_Turdidae_tree$tip.label), subtree = TRUE, cex = 0.8)
```


```{image} Practical_2_72_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```




```{image} Practical_2_72_1.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



This means that when collapsing a phylogenetic tree you run the risk of miss representing the relationship between the different genera. The only way to get round this is by:

a) making sure check to see how paraphyletic your tree is at the start; this can be done more easily by uploading and viewing your tree file @ https://itol.embl.de/. 
b) Renaming your conflicting paraphyletic clades within your phylogeny, by altering the individual species names.



```R
Ran_Turdidae_tree$tip.label[Ran_Turdidae_tree$tip.label=="Turdus_philomelos"]<-"Turdus1_philomelos"
```

### 6. Adding trait data to trees in `R`

Often basic tree plots in R are all you need for exploring data and your analysis. However, for publications and presentations it may be useful to plot trees with associated trait data. We will try plotting data with a tree, using the package `ggtree`, and extension of `ggplot2`.

For this exercise we will use the Turdidae tree (Thrushes) with some data on different habitat types.
Load in the data:


```R
turdidae_data <- read.csv("Turdidae_data.csv")
str(turdidae_data)
```

    'data.frame':	173 obs. of  3 variables:
     $ Jetz_Name: chr  "Alethe choloensis" "Alethe diademata" "Alethe fuelleborni" "Alethe poliocephala" ...
     $ Habitat  : chr  "Dense" "Dense" "Dense" "Dense" ...
     $ Body.Mass: num  41.3 31.5 52 32.4 35.2 ...


We first need to match our data to the tip labels. First we need to put an underscore in the names, and then match them to see if there's any name differences or missing species. In your coursework you will find that occasionally because of taxonomic disagreements, you might be missing species. The easiset way to solve this is by manually checking tips that don't match.


```R
# Replace the blank space with an underscore
turdidae_data$Jetz_Name <- turdidae_data$Jetz_Name %>% str_replace(" ", "_")
head(turdidae_data)
```


<table>
<caption>A data.frame: 6 × 3</caption>
<thead>
	<tr><th></th><th scope=col>Jetz_Name</th><th scope=col>Habitat</th><th scope=col>Body.Mass</th></tr>
	<tr><th></th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>Alethe_choloensis      </td><td>Dense    </td><td>41.30</td></tr>
	<tr><th scope=row>2</th><td>Alethe_diademata       </td><td>Dense    </td><td>31.54</td></tr>
	<tr><th scope=row>3</th><td>Alethe_fuelleborni     </td><td>Dense    </td><td>52.00</td></tr>
	<tr><th scope=row>4</th><td>Alethe_poliocephala    </td><td>Dense    </td><td>32.37</td></tr>
	<tr><th scope=row>5</th><td>Alethe_poliophrys      </td><td>Dense    </td><td>35.20</td></tr>
	<tr><th scope=row>6</th><td>Brachypteryx_hyperythra</td><td>Semi-Open</td><td>78.00</td></tr>
</tbody>
</table>



We'll use the `%in%` operator, which is useful checking if our species are in the tip labels


```R
turdidae_data$Jetz_Name %in% Ran_Turdidae_tree$tip.label
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class="list-inline"><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>FALSE</li><li>FALSE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li><li>TRUE</li></ol>



We can save the results, and use this to select the rows we need from turdidae_data. We'll use the `!` operator, which means NOT. So in this case it's the species that are not in the tip labels. This can also work for lots of other functions as well so try experimenting!


```R
index <- !(turdidae_data$Jetz_Name %in% Ran_Turdidae_tree$tip.label)
turdidae_data[index,]
```


<table>
<caption>A data.frame: 2 × 3</caption>
<thead>
	<tr><th></th><th scope=col>Jetz_Name</th><th scope=col>Habitat</th><th scope=col>Body.Mass</th></tr>
	<tr><th></th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>23</th><td>Chaetops_aurantius</td><td>Open</td><td>41.1</td></tr>
	<tr><th scope=row>24</th><td>Chaetops_frenatus </td><td>Open</td><td>45.6</td></tr>
</tbody>
</table>



One of them we renamed so we'll change that back for our plots. The others aren't in our taxonomy. This is because they've been moved to Chaetopidae, a new family of just rockjumpers. This often happens as there are three main bird taxonomies: Jetz, Birdlife, and American Ornithological Society. We'll just remove them from our dataset.


```R
# Change the name back
Ran_Turdidae_tree$tip.label[Ran_Turdidae_tree$tip.label=="Turdus1_philomelos"]<-"Turdus_philomelos"
# Get the species that ARE in the tips
index <- turdidae_data$Jetz_Name %in% Ran_Turdidae_tree$tip.label
# Select only these species
turdidae_data <- turdidae_data[index,]
```

Now we can drop the tips that don't match and get plotting!


```R
drop.tips <- setdiff(Ran_Turdidae_tree$tip.label, turdidae_data$Jetz_Name)

# Remove all the species except one per genus
turdi_tree <-drop.tip(Ran_Turdidae_tree, drop.tips)
plotTree(turdi_tree,ftype="i")
```


```{image} Practical_2_86_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



Now lets try using ggtree to plot our data with our phylogeny. First we need to install `ggtree`. This package was intentionally left out of the 'install.R' script. Because not all packages are available from `CRAN` directly through `R`, we'll install `BiocManager`. `devtools` is another great package for installing packages from github.


```R
install.packages("BiocManager")
BiocManager::install("ggtree")
```

    Installing package into ‘/usr/local/lib/R/site-library’
    (as ‘lib’ is unspecified)
    
    Bioconductor version 3.12 (BiocManager 1.30.10), R 4.0.3 (2020-10-10)
    
    Installing package(s) 'ggtree'
    
    Old packages: 'lubridate', 'magrittr', 'rprojroot', 'rstudioapi', 'vctrs',
      'foreign'
    



```R
library(ggtree)
library(ggplot2)
```

`ggtree` is a bit more complicated than just normal tree plots, but you can also do a lot more. We'll create a basic tree plot structure first and then add tip labels and traits after. We'll also make the plotting window bigger.


```R
# This resizes the plot windows if you use an online notebook. In RStudio you can do this when you export plots using jpeg() etc.
options(repr.plot.width=15, repr.plot.height=15)
turdidae_plot <- ggtree(turdi_tree, layout = "circular")
turdidae_plot
```


```{image} Practical_2_91_0.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



Now we'll create a simple one column dataframe with just the habitat data to plot and the species names as row names.


```R
habitat_data <- as.data.frame(turdidae_data[,2])
rownames(habitat_data) <- turdidae_data$Jetz_Name
colnames(habitat_data) <- "Habitat"
```

Now we can make our plot!


```R
# Create the heat map using habitat data. Offset tells us how far from the tips to plot the data and width is how wide the bars are.
gheat <- gheatmap(p=turdidae_plot, data=habitat_data, offset = 0.05, width=0.1, colnames = FALSE) + 
# Scale fill manual is a ggplot function. It lets the plot know what to 'fill' and what colours to use.
scale_fill_manual(name = 'Habitat', values=c("deeppink3", "goldenrod2", "cyan4")) + 
# Add the tip labels and offset them further from the bars
geom_tiplab(offset = 5, cex=4) + 
# We're adding tip labels on top of an existing plot, so we need to make the axes bigger so our labels don't fall off the edge.
xlim(0, 40) +
# This resizes the legend and makes the text bigger than the default. Try changing the numbers!
theme(legend.key.width = unit(0.8, "cm"), legend.key.height = unit(0.8, "cm"), legend.text = element_text(size = 13), legend.title = element_text(size = 13))

# Lastly plot the data!
plot(gheat)
```

    Scale for 'y' is already present. Adding another scale for 'y', which will
    replace the existing scale.
    
    Scale for 'fill' is already present. Adding another scale for 'fill', which
    will replace the existing scale.
    



```{image} Practical_2_95_1.png
:alt: plot
:class: bg-primary
:width: 700px
:align: center
```



And now we have a plot where we can see the spread of habitat types in Thrushes. Try experimenting with different colours and sizes to create some beautiful trees that put this one to shame! There's also lots of other ways you can label trees. For more info this guide is a great place to start: 
https://4va.github.io/biodatasci/r-ggtree.html#the_ggtree_package



> Extra task: Can you make another plot for continuous body mass data? Think about how you'd show this with colour. Can you change the key to better display the change in body mass? Is body mass more clumped in the tree than habitat? 
