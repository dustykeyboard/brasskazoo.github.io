---
layout: post
title: Add some color to your Jira logs!
date: '2011-12-21T15:27:00.001+11:00'
categories: software-development system-administration

author: brasskazoo
tags:
- log4j
- perl
- logging
- Jira
- Software Development
- tail
- unix
- tomcat
modified_time: '2012-01-02T19:24:10.141+11:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-7851881176689851182
blogger_orig_url: http://blog.brasskazoo.com/2011/12/add-some-color-to-your-jira-logs.html
---

Tired of the old black and white run-of-the-mill log files?
Well have I got the solution for you!

##     Whats the point?
This is a simple perl snippet that can help you quickly
identify important lines in your log files - specifically, I've made this for 
use for watching Atlassian Jira's tomcat logs, which uses the 
[log4j](http://logging.apache.org/log4j/) Java logging module. 

Since the log format is standardised in configuration files, its easy to write
regular expressions to match certain patterns or words within each line
([word-boundary](http://www.regular-expressions.info/wordboundaries.html)).

We want to surround the word with bash color codes to set and reset text color 
and backgrounds, such as `\e[1;33;41m` to set bright, yellow text with a red 
background and `\e[0m` to reset to defaults. 
The perl substitution would then look like this: 

````
s/\b(WARN)\b/\e[1;33;41m$&\e[0m/
````


You can see this in action quickly by running: 

````
cat {jira_home}/logs/catalina.out | perl -pe
"s/\b(WARN)\b/\e[1;33;41m$&\e[0m/"
````

##        Terminal Colors
I should note that I'm using the IR_Black theme on my
Mac OSX terminal. See [this page](http://blog.toddwerth.com/entries/6) for 
more info. Makes the colors a more pastel than you'll see below. 
You'll also need to have enabled colors in your bash profile.

The color code is made up of the following: 
1. Escape character '\e[' 
1. A. Normal or bright (0 or 1) 
1. B. Text color (30-37) 
1. C. Background color (40-47) 
1. lowercase 'm' to signify a color code 
In the form: 

````
\e[`A`;`B`;`C`m
````

The A, B and C sections are optional, but the '\e[' and trailing 'm' are 
required. See the following table for color combinations:

<table border="0">
<thead>
<th></th>
<th>Black (_0)</th>
<th>Red (_1)</th>
<th>Green (_2)</th>
<th>Yellow (_3)</th>
<th>Blue (_4)</th>
<th>Magenta (_5)</th>
<th>Cyan (_6)</th>
<th>White (_7)</th>
</thead>
<tr>
<th style="text-align: left;">Text (3_)</th>
<td
style="background: lightgrey; color: black;">30
<td style="background: lightgrey; color: red;">31
<td style="background: lightgrey; color: green;">32
<td style="background: lightgrey; color: yellow;">33
<td style="background: lightgrey; color: blue;">34
<td style="background: lightgrey; color: magenta;">35
<td style="background: lightgrey; color: cyan;">36
<td style="background: lightgrey; color: white;">37
   </tr><tr>
<th style="text-align: left;">Background (4_)</th>
<td style="background: black; color: lightgrey;">40
<td style="background: red; color: lightgrey;">41
<td style="background: green; color: lightgrey;">42
<td style="background: yellow; color: lightgrey;">43
<td style="background: blue; color: lightgrey;">44
<td style="background: magenta; color: lightgrey;">45
<td style="background: cyan; color: lightgrey;">46
<td style="background: white; color: lightgrey;">47</td>
</tr></table>

##     Converting to a bash function
We can use multiple substitution expressions by joining them with the
`||` operator. 
I've developed my function to highlight the keywords `DEBUG` (blue), `INFO` 
(green), `WARN` (yellow) and `ERROR` (red), and also highlight lines 
containing `SEVERE` and stacktraces in red (containing `Exception` or starting
with 'at'). 
The entire function looks like this: 

````
tailight() {
    tail -f -n 60 $1 \ 
    | perl -ple 's/\b(INFO)\b/\e[1;32m$&\e[0m/
    || s/\b(DEBUG)\b/\e[1;34m$&\e[0m/
    || s/\b(WARN)\b/\e[1;33m$&\e[0m/
    || s/\b(ERROR)\b/\e[1;31m$&\e[0m/
    || s/^(.*(SEVERE|Exception)|\sat\s).*/\e[1;31m$&\e[0m/'
}
````

An alternative for the `cat` command would be: 

````
catlight() {
    cat $1 \ 
    | perl -ple 's/\b(INFO)\b/\e[1;32m$&\e[0m/
    || s/\b(DEBUG)\b/\e[1;34m$&\e[0m/
    || s/\b(WARN)\b/\e[1;33m$&\e[0m/
    || s/\b(ERROR)\b/\e[1;31m$&\e[0m/
    || s/^(.*(SEVERE|Exception)|\sat\s).*/\e[1;31m$&\e[0m/'
}
````

##    Inspirations
1. [IR_Black Terminal Theme](http://blog.toddwerth.com/entries/6)
1. [Tail with color (UNIX discussion)](http://fixunix.com/unix/83044-tail-color.html)
1. [Color Your Tail With Perl](http://twoguysarguing.wordpress.com/2011/03/22/pro-tips-color-your-tail-with-perl/)
