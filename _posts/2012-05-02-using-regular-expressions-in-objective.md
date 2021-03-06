---
layout: post
title: Using Regular Expressions in Objective C
date: '2012-05-02T17:03:00.000+10:00'
categories: software-development iphone-development

author: brasskazoo
tags:
- regex
- Objective C
- Software Development
modified_time: '2012-06-13T12:51:58.614+10:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-5757842552263088177
blogger_orig_url: http://blog.brasskazoo.com/2012/05/using-regular-expressions-in-objective.html
---

As of iOS 4.0 and OSX 10.7, Apple provides a class `NSRegularExpression` which 
implements the use of regular expressions in Objective C. 

An immutable instance of `NSRegularExpression` is created
with a regex pattern and options, and upon execution, can create 
`NSTextCheckingResult` for each match. 
`NSTextCheckingResult` uses NSRange objects to describe the location of match 
groups within the source string - so that by applying the range to the source 
string, you can extract the resulting matched text.

## Code

The following code demonstrates a regex in action. The search
expression matches words starting with a vowel, and ending in 'd' or 's'. It 
will print the total number of matches, and then each group of each match. 
Output is below. 



{% highlight Objective-C %}
NSString *strSource = @"The NSRegularExpression class is used to represent and apply regular expressions to Unicode strings. An instance of this class is an immutable representation of a compiled regular expression pattern and various option flags. The pattern syntax currently supported is that specified by ICU.";

NSError *errRegex = NULL; 
NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:@"\\b(a|e|i|o|u)\\w+(s|d)\\b" options:NSRegularExpressionCaseInsensitive error:&errRegex];

NSUInteger countMatches = [regex numberOfMatchesInString:strSource options:0 range:NSMakeRange(0, [strSource length])];
NSLog(@"Number of Matches: %ld", countMatches); 

NSLog(@"-----"); 

[regex enumerateMatchesInString:strSource
                        options:0
                          range:NSMakeRange(0, [strSource length])
                     usingBlock:^(NSTextCheckingResult *match, NSMatchingFlags flags, BOOL *stop) {

    NSLog(@"Ranges: %ld", [match numberOfRanges]);

    NSString *matchFull = [strSource substringWithRange:[match range]]; 
    NSLog(@"Match: %@", matchFull); 

    for (int i = 0; i < [match numberOfRanges]; i++) {
        NSLog(@"\tRange %i: %@", i, 
            [strSource substringWithRange:[match rangeAtIndex:i]]);
    } 
}]; 

if (errRegex) { 
    NSLog(@"%@", errRegex); 
}
{% endhighlight %}


## Output
````
RegexTest[23743:403] Number of Matches: 4
RegexTest[23743:403] ----- 
RegexTest[23743:403] Ranges: 3 
RegexTest[23743:403] Match: used 
RegexTest[23743:403]  Range 0: used 
RegexTest[23743:403]  Range 1: u 
RegexTest[23743:403]  Range 2: d 
RegexTest[23743:403] Ranges: 3 
RegexTest[23743:403] Match: and 
RegexTest[23743:403]  Range 0: and 
RegexTest[23743:403]  Range 1: a 
RegexTest[23743:403]  Range 2: d 
RegexTest[23743:403] Ranges: 3 
RegexTest[23743:403] Match: expressions 
RegexTest[23743:403]  Range 0: expressions 
RegexTest[23743:403]  Range 1: e 
RegexTest[23743:403]  Range 2: s 
RegexTest[23743:403] Ranges: 3 
RegexTest[23743:403] Match: and 
RegexTest[23743:403]  Range 0: and 
RegexTest[23743:403]  Range 1: a 
RegexTest[23743:403]  Range 2: d
````

## Read More...
1. [Apple Developer Documentation](https://developer.apple.com/library/ios/#documentation/Foundation/Reference/NSRegularExpression_Class/Reference/Reference.html)