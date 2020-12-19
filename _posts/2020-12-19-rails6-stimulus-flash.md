---
layout: post
title: Minimal AJAX with Stimulus, Rails 6 and JS HTTP POST Request using Fetch
date: 2020-12-19 05:22 -0600
---
The process of adding AJAX calls to a Rails application generally results in adding a few hefty libraries to your apps especially if the result is a framework being added to handle dynamic user interactions. The Rails 6 + Stimulus + Turbolinks advertises a lightweight approach to provide dynamic user interactions and the advertising did lives up to its billing. Stimulus provides a very small footprint, short setup time, and straightforward view and controller wiring to add dynamic user interactions to your application.

Two areas caused a small bit of time loss in setup:
1. Setting up the folders and paths for stimulus
1. Creating an HTTP POST request (Using fetch and providing a CRSF Token)

The following will show a simple example and explicitly show the folder structure used and the function calls to provide the CSRF Token for a HTTP POST request to your Rails Controller.

## Stimulus Setup
The wiring of stimulus into the Rails 6 way of handling JavaScript did trigger a bit of research and the following seems to be a clean setup and that setup included added a controllers folder to the app/javascript folder to be the entry point for user actions in the browser.

First, use yarn or npm to add stimulus to your project:
```bash
yarn add stimulus
```

Add a _controllers_ directory to app/javascript.
In your app/javascript/packs/application.js import the _controllers_ directory:

```JavaScript
// app/javascript/packs/application.js
import "controllers"
```

Add an index.js to that directory with the following (reference stimulus docs for updates):

```JavaScript
//  app/javascript/controllers/index.js
import { Application } from "stimulus"
import { definitionsFromContext } from "stimulus/webpack-helpers"

const application = Application.start()
const context = require.context(".", true, /\.js$/)

application.load(definitionsFromContext(context))
```

## Stimulus MVC
The data- attributes used on HTML elements map provide a straightforward mapping and can be used in JavaScript controllers that will be created for the application.

### View
```html
<div class= "col" data-controller="message">
  <div>
    <div data-target="message.result"></div>
    <input data-target="message.content" type = "text">
    <button data-action="click->message#send">Send Message</button>
  </div>
</div>
```  

Note, the data_controller="message" will be wired to the controller _app/javascript/controllers/message_controller.js_.  The data-target DOM elements will be accessible in the controller.

### JavaScript Controller

The setup so far will get your request into the browsers JavaScript execution. The next step to do something in your application is to make a request to your backend server. There are several ways can do using jquery and other librarys and instead of adding more node modules I went with using the now include _fetch_ function. This seems like a good choice for this use case as there is minimal overhead. There was one issue that was run into.

app/javascript/controllers/message_controller.js
```JavaScript
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = ["content", "result"]

  async send() {
    const csrfToken = document.head.querySelector(`meta[name="csrf-token"]`).getAttribute("content")
    const messageContent = this.Content
    let resultMessage = ""

    let response = await fetch('/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json;charset=utf-8',
        'X-CSRF-Token': csrfToken
      },
      body: `{"message": "${this.contentTarget.value}"}`,
    });

    if (response.ok) {
      resultMessage = "Thank you for the message!"
    } else {
      console.log("HTTP-Error: " + response.status);
      resultMessage = "There was an error sending your message."
    }

    this.resultTarget.innerHTML = resultMessage
  }
}
```

### CSRF
The CSRF token is required, and should not be disabled, to successfully talk to your Rails Controller. Your Rails controller is still doing your heavy lifting. The initial attempts to talk to yoBefore the request successfully arrived to the rails server an error was encountered related to Cross Site Request Forgery.

```bash
ActionController::InvalidAuthenticityToken (ActionController::InvalidAuthenticityToken):
```

The previously rendered page has the CSRF Token available in the header and is accessible in from the Stimulus JavaScript controller. You will likely see the following error:

The following line will retrieve csrf-token from the rendered page:

```JavaScript
const csrfToken = document.head.querySelector(`meta[name="csrf-token"]`).getAttribute("content")
```

In the request you construct using fetch, provide that value as one of the header values:
```JavaScript
headers: {
  'Content-Type': 'application/json;charset=utf-8',
  'X-CSRF-Token': csrfToken
},

```

The addition of the csrf token will result in a successful POST request to your Rails controller and the ability to create and modify.

# Summary
This was a very quick and minimal, yet powerful, add to the application. The availability of the fetch method made the more minimal by not needing to include another library to initiate the HTTP Request back to the server. This approach seems to provide a great deal of runway for your application before moving to more complex solutions without sacrificing on the usability of your application.

# References
* https://stimulusjs.org/
* https://longliveruby.com/articles/rails-6-stimulus-js
* https://en.wikipedia.org/wiki/Cross-site_request_forgery
* https://blog.nvisium.com/understanding-protectfromforgery
