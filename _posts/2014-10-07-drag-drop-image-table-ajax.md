---
layout: post
title: Drag and Drop image over table
tags: [drag, drop, ajax, image, upload, table, list]
---

Suppose you are building some kind of shop with products list table. One of your use cases is uploading product images, here is nice way to do it modern way:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Drag &amp; Drop Images Example</title>
    <style>
        img {
            border: 0;
            display: block;

            max-width: 100px;
            height: auto;
        }

        table {
            border-collapse: collapse;
            border-spacing: 0;
        }

        td, th {
            padding: 5px;
        }

        @keyframes pulse {
            0% {
                transform: scale(1);
            }
            50% {
                transform: scale(0.9);
            }
            100% {
                transform: scale(1);
            }
        }

        @-webkit-keyframes pulse {
            0% {
                -webkit-transform: scale(1);
            }
            50% {
                -webkit-transform: scale(0.9);
            }
            100% {
                -webkit-transform: scale(1);
            }
        }

        tbody tr.active {
            background-color: #f7f7f7;
        }

        tbody tr.active img {
            -webkit-animation: pulse 2s infinite;
            animation: pulse 2s infinite;
        }

        pre {
            background: #f7f7f7;
            margin: 0;
            padding: .5em 1.5em;
        }

        details {
            margin: 2em 0;
        }

        summary {
            outline: none;
        }
    </style>
</head>
<body>
    <p>Here is dummy product list from shop, suppose that you want to change product image, just drop new image over it</p>
    <table>
        <caption>Product List</caption>
        <thead>
            <tr>
                <th>id</th>
                <th>image</th>
                <th>title</th>
                <th>price</th>
            </tr>
        </thead>
        <tbody>
            <tr data-id="1">
                <td>1</td>
                <td><img src="http://placehold.it/100x100"></td>
                <td>Macbook Air</td>
                <td>$ 999</td>
            </tr>
            <tr data-id="2">
                <td>2</td>
                <td><img src="http://placehold.it/100x100"></td>
                <td>Macbook Pro</td>
                <td>$ 1999</td>
            </tr>
            <tr data-id="3">
                <td>3</td>
                <td><img src="http://placehold.it/100x100"></td>
                <td>iPhone 6</td>
                <td>$ 1999</td>
            </tr>
        </tbody>
    </table>

<details>
<summary>handler.php could be something like this</summary>
<pre>&lt;?php

$id = $_POST['id'];
$src = $_FILES['image']['tmp_name'];
$dst = 'uploads/' . $id . '.' . pathinfo($_FILES['image']['name'], PATHINFO_EXTENSION);

try {
    if (move_uploaded_file($src, $dst)) {
        header('Content-type: application/json; charset=utf-8');
        echo json_encode(['id' =&gt; $id, 'image' =&gt; $dst]);
    } else {
        throw new Exception('There was error while moving uploaded file.');
    }
}
catch(Exception $ex) {
    http_response_code(500);
    header('Content-type: application/json; charset=utf-8');
    echo $ex-&gt;getMessage();
}</pre>
</details>

<script>
    function matches(el, selector) {
        return (el.matches || el.matchesSelector || el.msMatchesSelector || el.mozMatchesSelector || el.webkitMatchesSelector || el.oMatchesSelector).call(el, selector);
    }

    function closest(el, selector) {
        if(!el) return null;
        while(el.parentNode && !matches(el, selector)) el = el.parentNode;
        return el == document ? null : el;
    }
</script>
<script>
    var table = document.querySelector('table');

    table.addEventListener('dragenter', function(event) {
        event.stopPropagation();
        event.preventDefault();
        event.dataTransfer.dropEffect = 'copy';

        var node = closest(event.target, 'tr[data-id]');
        if(!node) return;

        node.className = 'active';
    });

    table.addEventListener('dragover', function(event) {
        event.stopPropagation();
        event.preventDefault();
        event.dataTransfer.dropEffect = 'copy';

        var node = closest(event.target, 'tr[data-id]');
        if(!node) return;

        node.className = 'active';
    }, false);

    table.addEventListener('dragleave', function(event) {
        event.stopPropagation();
        event.preventDefault();
        event.dataTransfer.dropEffect = 'copy';

        var node = closest(event.target, 'tr[data-id]');
        if(!node) return;

        node.removeAttribute('class');
    });

    document.querySelector('table').addEventListener('drop', function(event) {
        event.stopPropagation();
        event.preventDefault();

        var node = event.target;
        while(node.parentNode && !node.hasAttribute('data-id')) node = node.parentNode;
        if(!node.hasAttribute('data-id')) return;

        var image = [].filter.call(event.dataTransfer.files, function(file){
            return file.type.match('image.*');
        }).shift();
        if(!image) {
            node.removeAttribute('class');
            alert('Only images allowed here');
            return;
        }

        // Example sending image via ajax request
        // --------------------------------------
        // var data = new FormData();
        // data.append('id', node.getAttribute('data-id'));
        // data.append('image', image, image.name);

        // var request = new XMLHttpRequest();
        // request.open('POST', 'handler.php', true);
        // request.addEventListener('load', function() {
        //  if (this.status >= 200 && this.status < 400){
        //      var data = JSON.parse(this.response);

        //      var img = node.querySelector('img');
        //      img.src = data.image;

        //      node.removeAttribute('class');
        //  }
        // });
        // request.send(data);


        // Example draw local image
        var reader = new FileReader();
        reader.addEventListener('load', function(event){
            var img = node.querySelector('img');
            img.src = event.target.result;
        });
        reader.readAsDataURL(image);
        node.removeAttribute('class');

    }, false);
</script>
</body>
</html>
```


<iframe width="100%" height="500" src="http://jsfiddle.net/mac2000/q5jxLumq/1/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

<a href="http://jsfiddle.net/mac2000/q5jxLumq/1/">Sources on jsfiddle</a>

Also if you are using jquery example can be minified preatty well, but i was wondering to make it with native javascript.

Simple Generic Handler for ASP.NET:

```csharp
using System;
using System.Web;
using ImageResizer;
using ImageResizer.Util;

public class Upload : IHttpHandler
{
    public void ProcessRequest(HttpContext context)
    {
        context.Response.ContentType = "text/plain";

        //TODO: For this to work run: Install-Package ImageResizer
        //TODO: Read desired dimensions from request

        var imageJob = (new ImageJob(
            context.Request.Files["Image"],
            "~/Uploads/" + DateTime.Now.Year + "/" + DateTime.Now.Month + "/<guid>.<ext>",
            new Instructions("width=150&height=150&mode=crop"))
        {
            CreateParentDirectory = true
        }).Build();

        context.Response.Write(PathUtils.ResolveAppRelativeAssumeAppRelative(PathUtils.GuessVirtualPath(imageJob.FinalPath)));
    }

    public bool IsReusable
    {
        get
        {
            return false;
        }
    }
}
```