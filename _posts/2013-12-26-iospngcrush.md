---
layout: post
title: iOS Pngcrush
tags: ios
---

When building for iOS, things want to "optimize" PNGs for fast loading. Xcode ships with a hacked
version of pngcrush that adds the `-iphone` option to convert to the "CgBi" format. There are more
thorough writeups [elsewhere][cgbi], but in a nutshell, it pre-multiplies alpha and swaps the color
channels around so it can load things more quickly. The resulting file is dangerously close to a
PNG, but is illegal by the spec, so it's only useful on iOS.

So it offloads some of the load-time work to a build-time task to optimize things. But what it
*doesn't* do is a particularly good job of *shrinking* the file. In practical use on my current
work project, it actually *grows* the size of our art by about 8%. In comparison, when I ran
[crushpngs], after several minutes warming up the room, things had shrunk about 22%.

There's an interesting [case study on the topic][imageoptim]. In summary, the gain in reducing CPU
time decoding things is dwarfed by so much more IO (which jibes with my past experiences where disk
IO on an iDevice proved to be way *way* slower than I would have guessed for SSD). Their verdict:
crush your PNGs and ignore Apple's specific optimization; more IO for less CPU is a bad trade.

The thing that was conspicuously missing from that case study, though, was the possible double-win;
why just crush the file, when you could crush it *and* do Apple's magic? I hate the added build
complexity, but I'm all for shaving off all the load time I can. It turns out that the loader
handling the CgBi files doesn't handle the full PNG specification, so I wound up with files that
had all kinds of exciting visual artifacts depending how that particular stretch of image was
compressed. Oh well.

I'm kind of glad, in the end, since it keeps things simpler. There's just one pass of [crushpngs]
and all platforms get to ship the exact same assets. Now I just ned to get around to actually
integrating *that* into our build at some point.


[crushpngs]: https://github.com/deadmoose/crushpngs
[imageoptim]: http://imageoptim.com/tweetbot.html
[cgbi]: http://iphonedevwiki.net/index.php/CgBi_file_format
