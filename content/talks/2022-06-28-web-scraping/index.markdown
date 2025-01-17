---
title: "Web Scraping in R"
subtitle: "Webscraping using rvest and rselenium"
excerpt: "Tips and tricks to scrape text rvest and rselenium"
author: Pallav Routh
date: '2022-02-10'
slug: 
  - rvest
categories:
  - scraping
tags:
  - wrangling
---

I recently gave a talk on webscraping using R to faculty and students in my business school. This blog post is a summary of what I covered during that session. My main objective was to get users to understand the fundamentals of how to navigate a webpage and to how scrape element off of it. I did this by demonstrating a set of typical web scraping tasks.

The first task was to scrape certain texts from this [webpage](https://cran.r-project.org/web/views/Robust.html). Specifically, the first task was to extract the text from the first few paragraphs in the webpage.  The second task had two parts. The first part was to extract the text refering to the names of statistical methods. These are highlighted in bold (e.g. Linear, Non linear, Mixed-Effects). The second part was to extract the hyperlinks embedded within the text describing the above methods. 

In order to do these scraping tasks in R, we need to load the required packages -


```r
library(rvest)
library(magrittr)
```

```
## Warning: package 'magrittr' was built under R version 4.1.2
```

The very first thing we need to do when scraping is to read the webpage. In R, using the `rvest` package, we use the `read_html()` function to do that -


```r
url <- 'https://cran.r-project.org/web/views/Robust.html'
html <- read_html(url)
```

This function saves the entire html associated with the webpage. Now that we have the webpage, we need to find out which part of the html has the required items we need to scrape. We do this by opening the webpage in any standard browser and then inspecting the webpage in developer mode. 

Once we are in the developer mode, we hover over different elements of the html. This in turn highlights certain portions of the webpage. Once we locate the element we need, we can use the html tags to scrape the required portion. This will be more clear once we see an example. 

For the first task, we need to extract the first few paras. In developer mode, this corresponds to the text nestled within the `<p>...<\p>` tags. We can use the `html_nodes()` function in `rvest` to extract all elements associated with the `<p>...<\p>` tags. 


```r
p_tags <- html_nodes(html,css = "p")
(p_tags)
```

```
## {xml_nodeset (10)}
##  [1] <p>Robust (or “resistant”) methods for statistics modelling have been av ...
##  [2] <p>This task view is about R add-on packages providing newer or faster,  ...
##  [3] <p>Please send suggestions for additions and extensions via e-mail to th ...
##  [4] <p>An international group of scientists working in the field of robust s ...
##  [5] <p>We structure the packages roughly into the following topics, and typi ...
##  [6] <p><em><strong>Linear</strong> Regression:</em> <code>lmrob()</code> (<a ...
##  [7] <p>Note that a location (and scale) model is a regression with only an i ...
##  [8] <p><em><strong>Generalized</strong> Linear Models ( <strong>GLM</strong> ...
##  [9] <p><em><strong>Mixed-Effects</strong> (Linear and Nonlinear) Regression: ...
## [10] <p><em><strong>Nonlinear / Smooth</strong> (Nonparametric Function) Regr ...
```

We can also do this either using the xpath associated `<p>...<\p>` tags or the css tags. Below I show the outputs are the same -


```r
p_tagsx <- html_nodes(html,xpath = '//p')
all.equal(p_tags,p_tagsx)
```

```
## [1] TRUE
```

If we wanted to extract certain subelements of `p_tags` we repeat the process by re-using `html_nodes()`. For example, if we wanted the elements within the `<strong>...<\strong>` tags we would use -


```r
p_strong_tags <- html_nodes(p_tags,css = "strong")
(p_strong_tags)
```

```
## {xml_nodeset (7)}
## [1] <strong>Linear</strong>
## [2] <strong>Generalized</strong>
## [3] <strong>GLM</strong>
## [4] <strong>Mixed-Effects</strong>
## [5] <strong>mixed effects</strong>
## [6] <strong>LMM</strong>
## [7] <strong>Nonlinear / Smooth</strong>
```

Once we get to the nodes with the `<strong>...<\strong>` tags we can extract the text with the `html_text()` function. 


```r
html_text(p_strong_tags)
```

```
## [1] "Linear"             "Generalized"        "GLM"               
## [4] "Mixed-Effects"      "mixed effects"      "LMM"               
## [7] "Nonlinear / Smooth"
```

```r
html_text(p_strong_tags,trim = T)
```

```
## [1] "Linear"             "Generalized"        "GLM"               
## [4] "Mixed-Effects"      "mixed effects"      "LMM"               
## [7] "Nonlinear / Smooth"
```

In summary, the general tactic should be to get to the desired node using the `html_nodes()` function and then use the `html_text()` function to scrape the text. Below, I demonstrate this same principle on a different task. Here the task is to extract the links associated with every R package on the webpage. Also I demonstrate this example by using the css tags


```r
# accessing attributes - e.g. links encoded in hrefs
ul_tags <- html_nodes(html,css = "ul")
# accessing a particular node - see the object in R session
ul_tags[[3]]
```

```
## {html_node}
## <ul>
## [1] <li>We are <em>not</em> considering cluster-resistant variance (/standard ...
## [2] <li>“Truly” robust clustering is provided by packages <a href="../package ...
## [3] <li>See also the <a href="Cluster.html">Cluster</a> CRAN task view.</li>
```

Here you can use another handy function called `html_children()` to inspect the subdivisions of a particular node -


```r
# inspect further down the tree
html_children(html_nodes(ul_tags[[3]],css = "li"))
```

```
## {xml_nodeset (9)}
## [1] <em>not</em>
## [2] <a href="../packages/cluster/index.html">cluster</a>
## [3] <code>pam()</code>
## [4] <em>not</em>
## [5] <a href="../packages/genie/index.html">genie</a>
## [6] <a href="../packages/Gmedian/index.html">Gmedian</a>
## [7] <a href="../packages/otrimle/index.html">otrimle</a>
## [8] <a href="../packages/tclust/index.html">tclust</a>
## [9] <a href="Cluster.html">Cluster</a>
```

```r
html_nodes(ul_tags[[3]],css = "li a")
```

```
## {xml_nodeset (6)}
## [1] <a href="../packages/cluster/index.html">cluster</a>
## [2] <a href="../packages/genie/index.html">genie</a>
## [3] <a href="../packages/Gmedian/index.html">Gmedian</a>
## [4] <a href="../packages/otrimle/index.html">otrimle</a>
## [5] <a href="../packages/tclust/index.html">tclust</a>
## [6] <a href="Cluster.html">Cluster</a>
```

```r
# inspect
html_children(html_nodes(ul_tags[[3]],css = "li a")) #no more divisions
```

```
## {xml_nodeset (0)}
```

Once we get to the desired node ("li a" in this case), we can use the `html_attr()` function to extract the links -


```r
unlist(html_attrs(html_nodes(ul_tags[[3]],css = "li a")))
```

```
##                             href                             href 
## "../packages/cluster/index.html"   "../packages/genie/index.html" 
##                             href                             href 
## "../packages/Gmedian/index.html" "../packages/otrimle/index.html" 
##                             href                             href 
##  "../packages/tclust/index.html"                   "Cluster.html"
```

```r
href_tags <- html_attr(html_nodes(ul_tags[[3]],css = "li a"),"href")
# same thing using xpath... but be careful
html_nodes(html,xpath = "//ul[3]/li/a")
```

```
## {xml_nodeset (6)}
## [1] <a href="../packages/cluster/index.html">cluster</a>
## [2] <a href="../packages/genie/index.html">genie</a>
## [3] <a href="../packages/Gmedian/index.html">Gmedian</a>
## [4] <a href="../packages/otrimle/index.html">otrimle</a>
## [5] <a href="../packages/tclust/index.html">tclust</a>
## [6] <a href="Cluster.html">Cluster</a>
```

```r
href_tags <- html_attr(html_nodes(ul_tags[[3]],xpath = "//ul[3]/li/a"),"href")
```

A nice thing about Rvest is that we can use the pipe operator to create a 'tidy' version of the above set of operations -


```r
href_tags <-
  html %>%
  html_nodes(xpath = "//div/ul[3]/li/a") %>%
  html_attr("href")

# task : get all hrefs and follow then and get the titles (optional)
gethref <- function(x){
  hrefs <- x %>%
    html_nodes(css = "li a") %>%
    html_attr("href")
  if (length(hrefs) == 0){return(NA)} 
  else {return(hrefs)}
}
```

In the next blog post I will go over how to scrape ethically and use Rselenium to scrape information off dynamic webpages.
