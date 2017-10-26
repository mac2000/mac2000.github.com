---
layout: post
title: 100% Flash background

tags: [acrionscript, flash]
---

Зачем это нужно?
----------------

Для того чтобы на страничке был фон который растягивался бы на всю ширину страницы как обои на рабочем столе.

Код
---

```
stage.scaleMode = StageScaleMode.NO_SCALE;
stage.align = StageAlign.TOP_LEFT;
var wo:int = 0;
var ho:int = 0;
var ratio:Number;
var l:Loader = new Loader();
var p:String;

var paramObj:Object = getFlashVars();

if(paramObj.bg != undefined) p = paramObj.bg
else {
    p = "bg.jpg";
    trace('Background parameter is undefined. Setting default background "bg.jpg"');
}

var r:URLRequest = new URLRequest(p);
l.load(r);
l.contentLoaderInfo.addEventListener( Event.INIT , loaded)
var m:MovieClip = new MovieClip();
this.addChild(m);
function loaded(event:Event):void {
    m.addChild(l.content);
    wo = m.width;
    ho = m.height;
    ratio = getRatio(m);
    stageResizeHandler();
}

stage.addEventListener(Event.RESIZE, stageResizeHandler);

function stageResizeHandler(event:Event = null):void{

    m.width = stage.stageWidth;
    m.height = m.width / ratio;

    if(m.height < stage.stageHeight) {
        m.height = stage.stageHeight;
        m.width = m.height * ratio;
    }

    //centre
    m.x = (stage.stageWidth - m.width) * .5;
    m.y = (stage.stageHeight - m.height) * .5;
}

function getRatio(target:MovieClip):Number{
    var ratio:Number;
    target.width > target.height ? ratio = (target.width / target.height) : ratio = (target.height / target.width);
    return ratio;
}

function getFlashVars() {
    try {
        var paramObj:Object = LoaderInfo( this.root.loaderInfo ).parameters;
        return( paramObj );
    } catch ( e:Error ) {
        trace( "Error getting FlashVars" );
    }
}
```

Файлы: [bg](/images/wp/bg.zip)
