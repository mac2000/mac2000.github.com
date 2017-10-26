---
layout: post
title: jQuery, JSONp, Flickr, KnockoutJs example

tags: [flickr, jquery, json, jsonp, knockout, knockoutjs, ko]
---

```html
<!DOCTYPE HTML>
<html lang="en-US">
<head>
    <meta charset="UTF-8">
    <title>Flickr JSONp</title>
    <link href="http://twitter.github.com/bootstrap/assets/css/bootstrap.css" rel="stylesheet">
    <style>
        #flickr, #selected {
            -webkit-touch-callout: none;
            -webkit-user-select: none;
            -khtml-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
            -o-user-select: none;
            user-select: none;
        }

        .thumbnails li:nth-child(3n+1) {
            clear:left;
        }
        .thumbnail {
            background:#fff;
            position:relative;
            overflow:hidden;
            _zoom:1;
        }
        .thumbnail .buttons {
            visibility:hidden;
            position:absolute;
            right:6px;
            bottom:6px;
            opacity: 0;

            -moz-transition: opacity 0.3s linear;
            -webkit-transition: opacity 0.3s linear;
            -o-transition: opacity 0.3s linear;
            transition: opacity 0.3s linear;
        }
        .thumbnail:hover .buttons {
            visibility:visible;
            opacity: 1;
        }
    </style>
</head>
<body>
    <div class="container" style="margin:2em auto;">
        <div class="form-inline">
            <button data-bind="click: refreshPreviewImages" class="btn btn-primary">Refresh</button>
            <input type="text" data-bind="value: tags" placeholder="comma separated tags...">
            <select class="span1" data-bind="options: tagModeOptions, value: tagMode"></select>
        </div>
        <hr>
        <div class="row">
            <div class="span6">
                <div class="well">
                    <h4>Preview<span data-bind="visible: previewImages().length > 0"> (<span data-bind="text: previewImages().length"></span> images)</span></h4>
                    <hr>
                    <ul id="flickr" class="thumbnails" style="margin:auto" data-bind="foreach: previewImages">
                        <li>
                            <div class="thumbnail">
                                <img data-bind="click: $root.selectImage, attr: { src: thumb, alt: alt }">
                                <div class="buttons">
                                    <a data-bind="click: $root.selectImage" title="Add"><i class="icon-white icon-plus-sign"></i></a>
                                    <a data-bind="click: $root.previewImage" title="Preview"><i class="icon-white icon-info-sign"></i></a>
                                </div>
                            </div>
                        </li>
                    </ul>
                    <div data-bind="visible: previewImages().length == 0">No images were found</div>
                </div>
            </div>
            <div class="span6">
                <div class="well">
                    <h4>Selected<span data-bind="visible: selectedImages().length > 0"> (<span data-bind="text: selectedImages().length"></span> images)</span></h4>
                    <hr>
                    <ul id="selected" class="thumbnails" style="margin:auto" data-bind="foreach: selectedImages">
                        <li>
                            <div class="thumbnail">
                                <img data-bind="click: $root.removeImage, attr: { src: thumb, alt: alt }">
                                <div class="buttons">
                                    <a data-bind="click: $root.removeImage" title="Remove"><i class="icon-white icon-minus-sign"></i></a>
                                    <a data-bind="click: $root.previewImage" title="Preview"><i class="icon-white icon-info-sign"></i></a>
                                </div>
                            </div>
                        </li>
                    </ul>
                    <div data-bind="visible: selectedImages().length == 0">No images selected</div>
                </div>
            </div>
        </div>
        <hr>
        <pre><code data-bind="html: json"></pre>
    </div>

    <div id="preview" class="modal hide fade">
        <div class="modal-header">
            <a class="close" data-dismiss="modal" >&times;</a>
            <h3>Modal Heading</h3>
        </div>
        <div class="modal-body">
        </div>
        <div class="modal-footer">
            <a href="#" class="btn" data-dismiss="modal" >Close</a>
        </div>
    </div>

    <script src="http://code.jquery.com/jquery-latest.min.js"></script>
    <script src="http://twitter.github.com/bootstrap/assets/js/bootstrap-modal.js"></script>
    <script src="http://cloud.github.com/downloads/SteveSanderson/knockout/knockout-2.0.0.js"></script>
    <script>
        (function($){
            function Model() {
                var self = this;
                self.tags = ko.observable();
                self.tagModeOptions = ['all', 'any'];
                self.tagMode = ko.observable('all');
                self.selectedImages = ko.observableArray();
                self.previewImages = ko.observableArray();

                self.refreshPreviewImages = function(){
                    $.getJSON('http://api.flickr.com/services/feeds/photos_public.gne?jsoncallback=?', {
                            tags: self.tags(),
                            tagmode: self.tagMode(),
                            format: "json"
                        }, function(data){
                        var items = $.map(data.items, function(item){
                            var img = $('img', item.description);
                            return {
                                thumb: img.attr('src').replace('_m.jpg', '_t.jpg'),
                                middle: img.attr('src'),
                                full: img.attr('src').replace('_m.jpg', '.jpg'),
                                alt: img.attr('alt')
                            };
                        });

                        self.previewImages(items);
                    });
                };

                self.previewImage = function(data, e) {
                    $('#preview .modal-header h3').text(data.alt);
                    $('#preview .modal-body').html('<div style="text-align:center"><img src="'+data.middle+'" alt="'+data.alt+'" style="margin:auto;"></div>');
                    $('#preview').modal('show');
                    e.preventDefault();
                    return false;
                };

                self.selectImage = function(data, e) {
                    self.selectedImages.push(data);
                    self.previewImages.remove(data);
                    e.preventDefault();
                    return false;
                };
                self.removeImage = function(data, e) {
                    self.previewImages.push(data);
                    self.selectedImages.remove(data);
                    e.preventDefault();
                    return false;
                };

                self.json = ko.computed(function(){
                    return ko.toJSON(self.selectedImages());
                });
            }
            var model = new Model();
            ko.applyBindings(model);
            model.refreshPreviewImages();
        })(jQuery);
    </script>
</body>
</html>
```
