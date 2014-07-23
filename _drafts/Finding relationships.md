---
layout: post
title: "Data Cleaning - Detecting 1-1 and 1-M relationships"
published: true
---

While I have read and used lots of material around data cleaning – missing data, imputing data, long-to-wide, changing data types – in R, one more technique which has come handy is **_ascertaining 1-1 or 1-M relationship between different columns_** of your tabular data. Is a Member ID unique to each person (in most cases it is)? How about a Family ID? Can a person belong to 2 families? How about a person having two different ages in a dataset spanning a single year (Yes this happens!)?

One probable reason of not covering this could be that this is mostly domain specific – relationships will be dependent on the area you working in – and hence is expected to be clarified in the data dictionary of your data. However, most data dictionaries do not go beyond data types and field length.

Here I am going to provide scenarios to illustrate that establishing these relationships is a good – sometime essential – step during the exploration phase. Secondly I share R code, open to suggestions and clones.

###Scenario 1

My example here is from Healthcare Claims. Claims are sometimes submitted in a header-line format or sometimes in a consolidated format.

Below is the wide form of data – Procedure Codes being consolidated in a single row. In this case CLAIMID acts as primary key and just a check for uniqueness might suffice. Evidently each claim relates only to one member.

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>

In header-line format, Claims are broken into line items and ideally MEMBERID should be repeated. However look at row 5 onwards. Claim 3 has two different members. Of course such problems should be caught in data collection procedure (or if wide data is converted to long form, the system doing so will be responsible). 

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>

Even with this trivial example, it is clear that other data cleaning exercises – data type check, missing value check – will not catch this and hence the need to establish the relationships.

###Scenario 2

Most of the healthcare data I work with is de-identified to address the privacy concerns. Here I present to you a peculiar example – this data was devoid of any dates including date of birth.

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>

Check the age for Member01. Two different ages. If we proceed with any age distribution analysis, we will end with faulty counts. Multiple approaches can used to overcome this like binning, etc. However the first step is to identify this. 

Of course the root cause of the problem was available (which I missed) in the data dictionary. It read – “AGE field denotes the age of the member on the date of service” and hence the different ages are actually one before and one after the member’s birthday. Again it is important to establish the relationship to identify this data issue.

As I mentioned earlier, these relationships are supposedly domain specific, and established practices, if followed, give you a good idea of what to expect. So you expect 1-1 relationship between Member IDs and Member Names. You might, by experience, might not expect this of Member Address. Multiple addresses are possible like Office and Residence Address.

Format fragmentation, however, is a ubiquitous problem. Different Standard Committees come up with different formats and you are left handling different representations of ~~same~~ similar data.

###R Code
Here's the code. First up one line code to check relationships in interactive mode.
{% highlight r %}
```R
# data.table example
# data.table[,.N,list(Col1,Col2)][,.N,Col1][N>1]
# Should Return 0 rows for 1-X Col1-Col2 relationship. Run by changing Col1 to Col2 to check reverse
# plyr example for data.frame
# ply()
# Scenario 1 above
claim<-data.frame()
```
{% endhighlight %}
And below is above wrapped in a function so that it can be applied to entire data. Functions default to checking relationships for all columns, which can be overridden by passing specific column names as a vector. **The function will create a data table copy.**
{% highlight r %}
```R
find.rel <- function(dt,names=NULL){
  library(data.table)
  library(reshape2)
  if(!is.data.table(dt)){ dt<-data.table(dt) }
  f1 <- function(col1,col2){
    cols <- c(col1,col2)
    nrow(dt[,.N,by=cols][,.N,by=col1][N>1])==0
  }
  if(is.null(names)) { names <- names(dt)[sapply(dt, is.factor)] }
  final <- CJ(col1=names,col2=names)
  final <- final[final$col1<final$col2]
  final[,rel1to2:=f1(col1,col2),list(col1,col2)]
  final[,rel2to1:=f1(col2,col1),list(col2,col1)]
  final[,relation:=paste(ifelse(rel1to2,"One","Many"),"to",ifelse(rel2to1,"One","Many"))]
  dcast(final[,list(col1,col2,relation)],col1~col2,value.var="relation")
}

find.rel(iris)
#Output

#Visual Output - @todo Heat map  ggplot2
```
{% endhighlight %}
As the function is implemented for data.table, I would like to request clones for data.frame, implemented in plyr, dplyr, etc. Any other comments are also welcome [here](http://saurabhagur.com)