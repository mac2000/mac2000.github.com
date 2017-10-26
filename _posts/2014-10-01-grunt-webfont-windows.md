---
layout: post
title: grunt-webfont on windows
tags: [grunt, webfont, svg, icons, windows, vagrant, virtualbox]
---

Unfortunatelly all modern ways to create icon fonts are not forking under windows, but they can still be used.

Here is tricky way: we will use virtual machine with linux to run tools, the good thing is that this virtual machine is provisioned so can be up and running very fast, also you do not need to install 100500 of programs to your operating system.

Requirements: Virtualbox, Vagrant

**Vagrantfile**

```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/trusty64"
    config.vm.provision :shell, path: "Provision.sh"
end
```

**Provision.sh**

```sh
#!/usr/bin/env bash

# Use closest mirrors available
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt trusty main restricted universe multiverse" | sudo tee --append /etc/apt/sources.list
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt trusty-updates main restricted universe multiverse" | sudo tee --append /etc/apt/sources.list
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt trusty-backports main restricted universe multiverse" | sudo tee --append /etc/apt/sources.list
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt trusty-security main restricted universe multiverse" | sudo tee --append /etc/apt/sources.list

# Node JS
sudo add-apt-repository -y ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install -y nodejs fontforge

# ttfautohint - there is trouble with 0.9.7
# *** Error in `ttfautohint': malloc(): memory corruption: 0x000000000153de20 ***
# so we need to install new one from sources instead of packages
sudo apt-get install -y autoconf automake bison flex git libtool perl build-essential libharfbuzz-dev pkg-config libfreetype6-dev libfreetype6

git clone git://repo.or.cz/ttfautohint.git
cd ttfautohint
git checkout v1.1
./bootstrap
./configure --with-qt=no --with-doc=no
make
sudo make install
cd ..
rm -rfv ttfautohint/
sudo ln -s /usr/local/bin/ttfautohint /usr/bin/


sudo npm install --global grunt-cli

ln -s /vagrant/package.json /home/vagrant/
ln -s /vagrant/Gruntfile.js /home/vagrant/

cd /home/vagrant

npm install
grunt
```


This two files will create and provision virtual machine and install all required software on it.

**package.json**

```json
{
    "name": "iconfont",
    "version": "1.0.0",
    "dependencies": {},
    "devDependencies": {
        "grunt": "^0.4.5",
        "grunt-webfont": "^0.4.8"
    }
}
```

**Gruntfile.js**

```js
module.exports = function(grunt) {

    grunt.initConfig({
        webfont: {
            icons: {
                src: '/vagrant/svg/*.svg',
                dest: '/vagrant/icons',
                options: {
                    font: 'ri',
                    syntax: 'bootstrap',
                    //stylesheet: 'scss',
                    relativeFontPath: '//css.rabota.com.ua/icons',
                    htmlDemoTemplate: '/vagrant/template.html',
                    templateOptions: {
                        classPrefix: 'ri_',
                        mixinPrefix: 'ri_'
                    }
                }
            }
        }
    });

    grunt.loadNpmTasks('grunt-webfont');

    grunt.registerTask('default', ['webfont']);
};
```

**template.html**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title><%= fontBaseName %></title>
    <style>
        html {font-family: sans-serif}

        body {margin: 2em auto; max-width: 60em; background:#F7F7F7; color:#333333}

        ul {
            list-style: none; padding: 0; margin: 0;

            -webkit-column-count:5; -moz-column-count:5; column-count:5;
            -webkit-column-gap:20px; -moz-column-gap:20px; column-gap:20px;
        }
        li {line-height:2em}
        i {display:inline-block; width:32px; text-align:center; font-size: 22px}

        <%= styles %>
    </style>
</head>
<body>
    <ul class="icons" id="icons">
        <% for (var glyphIdx = 0; glyphIdx < glyphs.length; glyphIdx++) { var glyph = glyphs[glyphIdx] %>
        <li>
            <i class="<%= classPrefix %><%= glyph %>"></i>
            <%= classPrefix %><%= glyph %>
        </li>
        <% } %>
    </ul>
</body>
</html>
```

This last one is optional, if you want something cleaner than default one.
