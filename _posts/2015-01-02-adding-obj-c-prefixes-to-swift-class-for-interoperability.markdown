---
layout: post
title: "Adding Obj-C Prefix to a Swift Class for Interoperability"
modified:
categories: 
excerpt: Understanding how to use class prefixes with Swift classes in Objective C code.
tags: [iOS, Objective-C, Swift]
image:
  feature:
date: 2015-01-02T15:28:21-08:00
comments: true
---

#Background

Apple has provided a tonne of documentation on using Swift and Objective C in the same project, however this issue had me very confused for a while. 

I wanted my nice, clean, prefixless Swift class to be used in my Objective C code, but I wanted a prefix to keep in line with Obj-C best practices.

The Apple docs state:

> Swift also provides a variant of the @objc attribute that allows you to specify name for your symbol in Objective-C.

and provide the following example:

{% highlight swift %}
@objc(Squirrel)
class Белка {
    @objc(initWithName:)
    init (имя: String) { /*...*/ }
    @objc(hideNuts:inTree:)
    func прячьОрехи(Int, вДереве: Дерево) { /*...*/ }
}
{% endhighlight %}

#The Problem

I specified my nice prefix as suggested in the `@objc` tag.

{% highlight swift %}
@objc(PHDeleteCell)
class DeleteCell {
	/*...*/
}
{% endhighlight %}

However my Objective C code couldn't seem to find the prefixed class, and insisted I use `DeleteCell` as the class name.

Looking at the generated `ModuleName-Swift.h` header file, the generated Obj-C header is as follows:

{% highlight objc %}
SWIFT_CLASS("PHDeleteCell")
@interface DeleteCell : UITableViewCell
- (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier OBJC_DESIGNATED_INITIALIZER;
- (instancetype)initWithCoder:(NSCoder *)aDecoder OBJC_DESIGNATED_INITIALIZER;
@end
{% endhighlight %}

#The Solution

Well, it's not so much of a solution, but an answer.

I used the suggested `DeleteCell` class name in my Objective C code, and built and ran the project. I then set a breakpoint in the debugger and inspected the cell after it was created, which showed the following:

{% highlight objc %}
(lldb) po cell
<PHDeleteCell: 0x7fa81a74ad10; baseClass = UITableViewCell; frame = (0 374.13; 375 49); autoresize = W; layer = <CALayer: 0x7fa81a74d9d0>>
{% endhighlight %}

Turns out, that *at runtime the class prefix is used*. 

Which means I **should** use the prefixless class name in my Objective C code, and don't have to worry about a conflict.
