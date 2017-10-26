---
layout: post
title: Flash Action Script 3 desaturate image

tags: [actionscript, actionscript3, as, as3, desaturate, flash, image]
---

For image desaturation use this code:

```
var matrix:Array = new Array(0.309, 0.609, 0.082, 0, 0, 0.309, 0.609, 0.082, 0, 0, 0.309, 0.609, 0.082, 0, 0, 0, 0, 0, 1, 0);
var cmFilter:ColorMatrixFilter = new ColorMatrixFilter(matrix);
myMovieClip.filters = [cmFilter];
```

Notice, u desaturating movie clip - so image must be loaded into it

Used for </110/>
