---
layout: post
title: "Rendering NSTextAttachment inline in a UITextView"
modified:
categories: 
excerpt: "Getting text attachments to behave well with font baselines, without getting buried in TextKit."
tags: []
image:
  feature:
date: 2014-12-19T17:32:09-08:00
comments: true
share: true
---

#Background

Apple's Mail app for iOS seems to treat a single email address as a single character once entered. When you try to edit the text, the whole email is selected.

![Email selecting]({{ site.url }}/images/Screen Shot 2014-12-19 at 8.55.14 PM.png)

In an attempt to mimic the email entry behaviour of the Mail app, I came upon an issue when rendering instances of `NSTextAttachment` inside a `UITextView`.

#The Problem

To accomplish this, I attempted to take snapshots of the entered text, and render it as a text attachment inline with any additional text (I might write another post about the data source behaviour another time).

The technique worked very well, except I found my text baseline was being thrown around a bit.

![NSTextAttachment Baseline]({{ site.url }}/images/Screen Shot 2014-12-19 at 9.02.50 PM.png)

This had me stumped for a while. The `UIImage` that I had generated for the `NSTextAttachment` was simply a `UILabel` rendered offscreen, and then sized to fit. Given that all my font sizes and string attributes matched up, there should not be any reason for the baselines to not line up *perfectly*.

I set a background colour on the label to investigate:

![Background colour showing baseline]({{ site.url }}/images/Screen Shot 2014-12-20 at 1.02.48 AM.png)

![Close up of descenders]({{ site.url }}/images/Screen Shot 2014-12-20 at 1.03.06 AM.png)

Aha! It looks like by default, the `UITextView` renders instances of `NSTextAttachment` on the *baseline*, extending upwards. This means the larger the image, the further down my baseline of the following text will be pushed.

You can see the snapshotted `UILabel` has space at the bottom for it's descenders, so we don't want it rendering on the baseline.

#The Solution

After spending far too much time delving into various TextKit classes, I came up with a fairly straightforward solution.

I started checking out the `NSLayoutManager` class, as that seemed to control how glyphs and characters are positioned in a text container. However it turns out that the `NSTextAttachment` class already has a method returning it's bounds:

{% highlight objc %}
-attachmentBoundsForTextContainer:proposedLineFragment:glyphPosition:characterIndex:
{% endhighlight %}

So, I made a simple subclass of `NSTextAttachment` which exposes a property for adjusting the y-offset of the attachment bounds.

{% highlight objc %}
@interface InlineTextAttachment : NSTextAttachment

@property CGFloat fontDescender;

@end

@implementation InlineTextAttachment

- (CGRect)attachmentBoundsForTextContainer:(NSTextContainer *)textContainer proposedLineFragment:(CGRect)lineFrag glyphPosition:(CGPoint)position characterIndex:(NSUInteger)charIndex {
	CGRect superRect = [super attachmentBoundsForTextContainer:textContainer proposedLineFragment:lineFrag glyphPosition:position characterIndex:charIndex];
	superRect.origin.y = self.fontDescender;
	return superRect;
}

@end
{% endhighlight %}
	
Using this subclass now, I can grab the descender value from the font I'm using in my attributed string.

{% highlight objc %}
UIImage *labelImage = [label snapshotImageAfterScreenUpdates:NO];
InlineTextAttachment *attachment = [[InlineTextAttachment alloc] initWithData:nil ofType:nil];
UIFont *font = [aString attribute:NSFontAttributeName atIndex:0 effectiveRange:NULL];

attachment.fontDescender = font.descender;
attachment.image = labelImage;
{% endhighlight %}

And bingo! The descender height was exactly the offset I needed to get all my `NSTextAttachment` instances to line up nicely with my existing text.

![Final result]({{ site.url }}/images/Screen Shot 2014-12-20 at 12.48.50 AM.png)
