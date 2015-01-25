---
layout: post
title: "SDWebImage : NSURLErrorDomain - Code=406"
date: 2015-01-25 15:27:51 +0530
comments: true
categories: 
---

The iOS project that I am working on, has decided to use some caching mechanism to improve perfomance and save bandwidth of the user. So as per the recommendation of some blogs and popularity, we have decided to go with SDWebImage cache. After inspecting example code and trying out some sample code I thought it'll be good to use.

But it wasn't work with images those are retrieved from our sever. I have hardcoded public images found from the image search. There wasn't any problem with those. I tried with different methods provided from the API. But the result is the same.

I am getting following error instead of successful response.

> Error Domain=NSURLErrorDomain Code=406 "The operation couldn’t be completed. (NSURLErrorDomain error 406.)"

Following is the definition I found from the Wikipedia for HTTP status code 406.

> 406 Not Acceptable - The requested resource is only capable of generating content not acceptable according to the Accept headers sent in the request.

I couldn't find any satisfiable answers from any Q/A sites. So I have decided to read the source code and identify any possible problem / solution.

After going through several classes, I found that `SDWebImageDownloader` class has a code related to setting the `'Accept'` header value into the request.

{% codeblock lang:objc %}
_HTTPHeaders = [NSMutableDictionary dictionaryWithObject:@"image/webp,image/*;q=0.8" 
												  forKey:@"Accept"];
{% endcodeblock %}
        
So above code caused me the problem. Once I changed it, caching started to work and work nicely. But that was a temporary solution, because it's not good to change library source code. 

I was using `UIImageView` category provided by the library to retrieve and cache images. So it was problematic to set header values from there. Also only `SDWebImageDownloader` class providing the necessary API method to change the header value for public users. So at first sight I was so disappointed and thought of trying something else. But luckily `SDWebImageDownloader` is a singleton object. Every method is using the `SDWebImageDownloader downloadImageWithURL:options:progress:completed:` method to download images. So I have set a custom header value to `'Accept'` header to override the default header.

{% codeblock lang:objc %}
[SDWebImageDownloader.sharedDownloader setValue:@"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8"
                             forHTTPHeaderField:@"Accept"];
[self sd_setImageWithURL:[self imageUrlForId:imageId] placeholderImage:image];
{% endcodeblock %}    