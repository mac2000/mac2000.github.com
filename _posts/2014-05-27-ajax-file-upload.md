---
layout: post
title: Ajax File Upload
tags: [ajax, xhr, file, upload]
---

http://blog.teamtreehouse.com/uploading-files-ajax

**handler.php**

```php
<?php
if(strpos($_FILES['file']['type'], 'image') !== 0) die('INVALID');
echo move_uploaded_file($_FILES['file']['tmp_name'], __DIR__ . DIRECTORY_SEPARATOR . $_FILES['file']['name']) ? 'OK' : 'ERROR';
```

**index.html**

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Ajax File Upload</title>
    <style>
        form > div {
            display: flex;
        }

        label {
            order: 1;
            margin-top: .2em;
            margin-right: .5em;
        }

        input {
            order: 2;
        }

        input:required + label:after {
            content: '*';
            color: red;
        }

        input:valid + label {
            color: green;
        }
    </style>
</head>
<body>
    <form id="form" action="handler.php" method="POST" enctype="multipart/form-data">
        <div>
            <input title="Select your photo" type="file" name="file" id="file" required>
            <label for="file">Select your photo</label>
        </div>
        <input type="submit" id="submit" value="Submit">
    </form>
    <script>
        var form = document.getElementById('form'),
            file = document.getElementById('file'),
            submit = document.getElementById('submit');

        file.onchange = function(event) {
            file.setCustomValidity(file.files[0].type.match('image.*') ? '' : 'INVALID');
        }

        form.onsubmit = function(event) {
            event.preventDefault();

            if (!form.checkValidity()) return false;

            var formData = new FormData(),
                xhr = new XMLHttpRequest();

            submit.innerHTML = 'Uploading...';
            submit.setAttribute('disabled', 'disabled');
            file.setAttribute('disabled', 'disabled');

            // Files: formData.append(name, file, filename);
            // Blobs: formData.append(name, blob, filename);
            // Strings: formData.append(name, value);
            formData.append('file', file.files[0], file.files[0].name);

            xhr.open('POST', 'handler.php', true);
            xhr.onload = function () {
                alert(this.response);
                submit.innerHTML = 'Submit';
                submit.removeAttribute('disabled');
                file.removeAttribute('disabled');
            };
            xhr.send(formData);
        }
    </script>
</body>
</html>
```