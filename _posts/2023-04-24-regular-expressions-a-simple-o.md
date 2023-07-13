---
author: Erwin Timmerman
header_img: _posts/img/2023-04-24-regular-expressions-a-simple-o.png
source: linkedin
source_file: linkedin_2023-04-24-T07-13-34
source_url: https://www.linkedin.com/pulse/regular-expressions-simple-overview-erwin-timmerman/
subtitle: "If you want to find/replace text automatically, regular expressions can\
  \ be of great help to search for specific parts or patterns in a text. Here are\
  \ some examples to get you started. \n  Character sets \n   \n    \n   \n  White\
  \ space \n   \n    \n   \n  Repetition \n   \n    \n   \n  Position in the line\
  \ \n   \n "
tags: []
title: Regular expressions - a simple overview

---
![Regular expressions - a simple overview]({{site.cdn_img_raw}}/_posts/img/2023-04-24-regular-expressions-a-simple-o.png)

If you want to find/replace text automatically, regular expressions can be of great help to search for specific parts or patterns in a text. Here are some examples to get you started.


## Character sets



![No alt text provided for this image]({{site.cdn_img_raw}}/_posts/img/1560340132224.png)

## White space



![No alt text provided for this image]({{site.cdn_img_raw}}/_posts/img/1559819902146.png)

## Repetition



![No alt text provided for this image]({{site.cdn_img_raw}}/_posts/img/1559820400518.png)

## Position in the line



![No alt text provided for this image]({{site.cdn_img_raw}}/_posts/img/1559820441775.png)

## Groups



![No alt text provided for this image]({{site.cdn_img_raw}}/_posts/img/1559820457383.png)

## Look ahead / Look back



![No alt text provided for this image]({{site.cdn_img_raw}}/_posts/img/1559820519474.png)

## Practical examples


Regular expressions can form equations that may look quite complex, but when you break them down, it is easier to understand them. Here are a few examples that show the power of regular expressions.


### Find numbers that have 3 or more repeated digits, such as 3*111* or 5*999*2.


You can use this, for example, to check if someone does not enter a PIN that is too simple.



![No alt text provided for this image]({{site.cdn_img_raw}}/_posts/img/1560340132573.png)

### Replace the mm-dd-yy date format with the dd-mm-yy date format, and vice versa.


This function replaces, for example, **9-17-’69** with **17-9-’69**, and **17/05/1973** with **05-17-1973**.



![No alt text provided for this image]({{site.cdn_img_raw}}/_posts/img/1560340132652.png)

### Remove manual formatting from HTML code.


This function removes all spans with styles (instead of classes) from HTML code.



![No alt text provided for this image]({{site.cdn_img_raw}}/_posts/img/1560340132605.png)

## Flavors


There are several flavors of regular expressions. Most of these examples work on all of them, but some might need a different syntax in specific programming languages. If an example does not work as expected, look at the documentation of your specific application.


## Want to know more?


For a more detailed explanation of regular expressions, have a look at these websites:


* https://www.regular-expressions.info/quickstart.html
* https://www.rexegg.com/regex-quickstart.html


