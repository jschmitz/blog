---
layout: post
title: d3 JS on Rails 6
---

It took me a bit of time to figure out how so I thought I'd share by showing the installation and config process and a simple script to use to demonstrate all the plumbing is setup correctly. From there it should be off to the races to all of d3 awesomeness.

I started with a vanilla Rails 6 app installed and will use webpacker.

## Install and require d3

First use yarn to install d3.
```bash
yarn install d3
```

In your rails app open up the file _app/javascript/packs/application.js_

_app/javascript/packs/application.js_

``` JavaScript
require("d3")
```

D3 is ready to use. Next we'll write a bit of JavaScript using a bit of D3 sample code and demonstrate it really is ready to use.

## Add a JavaScript Pack
 _app/javascript/packs/my_app.js_
``` javascript
require("packs/my_app")
```

_app/javascript/packs/my_app.js_
``` javascript
import * as d3 from "d3"

export function paint() {
  d3.select("body").style("background-color", "green");
}

document.addEventListener("turbolinks:load", function() {
  paint();
});
```

Load any page any page in your rails app and you should now have a great looking green background.

Have fun building awesome visualizations with d3!
