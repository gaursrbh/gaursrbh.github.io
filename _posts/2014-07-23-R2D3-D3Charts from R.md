## D3 Charts from R

Many attempts have already been made to generate D3 charts from R. Most notably [rCharts by Ramnath V](http://rcharts.io). 

Still in trying to hide D3 JS intricacies and provide functions to tinker with all chart attributes, rCharts has set a very high goal for itself and rightly so, as it probably aims to cater to R crowd with little experience in JS.

I believe that in this goal of bridging R and D3, R's role should be data preparation alone( which it already does well with reshape2, plyr and JSON-related packages). All chart related config and interactive actions should be left to D3.

The most important aspect is combining the data with D3 and generating charts from R Console. As used in rCharts backend, whisker package is the solution here using moustache templating system.

I am prosposing the following workflow:
1. Prepare your D3 templates. One time activity. Get your hands dirty with D3. Chalk out your layout, axes, etc. Bring in the interactivity - tool tips, zooming, etc.
2. Every time you need to generate the chart, prepare your data in R. Over time, the commands you use could be turned into functions.
3. Combine your data with the template using whisker. Voila! you have your D3 chart

Here's how it works

{% highlight r %}
library(RCurl)
source_https <- function(url, ...) {
  # parse and evaluate each .R script
  sapply(c(url, ...), function(u) {
  eval(parse(text = getURL(u, followlocation = TRUE, cainfo = system.file("CurlSSL", "cacert.pem", package = "RCurl"))), envir = .GlobalEnv)
  })
}

#Prepare the data
source_https("https://raw.github.com/gaursrbh/R2D3/master/SunBurst/hcjson.R")
data<-iris
cluster<-hclust(dist(data))
final<-toJSON(hcjson(cluster$merge))
write(final,file="datafile.json")
params<-list(jsonpath="datafile.json")

#Fetch template and pass the parameters
library(whisker)
#Template prepared beforehand
template<-getURL("https://raw.githubusercontent.com/gaursrbh/R2D3/master/SunBurst/template.html")
write(whisker.render(template, params),file="output.html")
{% endhighlight %}

Welcome your comments on Twitter [@srbhgaur](https://twitter.com/srbhgaur)

