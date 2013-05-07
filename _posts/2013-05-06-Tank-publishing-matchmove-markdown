---
date: 2013-01-22 8:53AM
layout: post
published: true
title: "Python <small>generating a simple overlay histogram for an image</small>"
synopsis: "Using pylab we can extract the values for each of the channels and create an overlay graph which we can then render on top of the original image."
tags:
- python
- PIL
- pylab
- image
---

* This will become a table of contents (this text will be scraped).
{:toc  class="nav nav-list well"}

### Backstory

This little example came about following a request from a supervisor who was remembering the 'old' days of when they used to use Shake and how they used to have a little histogram overlay which they could apply over the top of their comp during tech checks to quickly see if there where were any illegal values or obvious clipping occurring.

Unfortunately Nuke doesn't have anything out of the box which quite matches this behaviour, and whilst the histogram Node is lacking, it is not a big enough bug bear to cause anyone to really want to ever launch Shake again.

*That is a minor lie, the 2D tracker definitely is still good enough when compared to the one in Nuke that we still catch Compositors firing it up for a quick track.*

### Example

![](https://raw.github.com/andrewbunday/PyHistogram/master/reference/example.png)

### Quick Explanation

So here's whats happening:

1. PIL.Image is used to open the input image, in this case a jpg but other formats could be used, and the Image class replaced with a reader for your particular image. The only important feature of the reader class is that it must be capable of formatting the pixel data as an array compatible with pylab.array().
2. Each of the channels is extracted from the image for separate manipulation.
3. We start a new figure, this initialises pylab.
4. For each channel we compute a histogram of the pixel values. These will be displayed, overlaid on the figure, but will not be scaled to fit nicely within the bounds of our reference image.
5. We compute the scale to apply to the x/y axis by finding out the maximum number of samples for each channel (X) and the maximum value of each sample (Y).
6. All of the image data is displayed, and we modify the labels and properties of the graph axis to improve their legibility. 

### The Code

{% highlight python linenos %}
from PIL import Image
import pylab
import os

file_path = '%s/../reference/aa_van_and_bike.jpg' % os.path.dirname(os.path.abspath(__file__))
ncolors = 255

im = pylab.array(Image.open(file_path))

blue  = [ b for channel in im[:,:,2] for b in channel ]
green = [ g for channel in im[:,:,1] for g in channel ]
red   = [ r for channel in im[:,:,0] for r in channel ]

pylab.figure()
b_n, b_bins, var = pylab.hist(blue, ncolors, histtype='stepfilled', align='left',alpha=0.5,aa=False,ec='none')
g_n, g_bins, var = pylab.hist(green, ncolors, histtype='stepfilled', align='left',alpha=0.5,aa=False,ec='none')
r_n, r_bins, var = pylab.hist(red, ncolors, histtype='stepfilled', align='left',alpha=0.5,aa=False,ec='none')

max_samples = pylab.amax([len(b_n), len(g_n), len(r_n)])
max_density = pylab.amax([b_n.max(), g_n.max(), r_n.max()])
pylab.imshow(im, interpolation='bilinear', extent=[0,max_samples,0,max_density])

pylab.axis('tight')
axes = pylab.gca()
axes.get_xaxis().set_ticks([])
axes.get_yaxis().set_ticks([])
pylab.xlabel('Saturation (0-1)')
pylab.ylabel('Density')

pylab.title('Color Density Analysis of %s' % os.path.basename(file_path))
pylab.show()
{% endhighlight %}