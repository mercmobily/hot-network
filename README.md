Too many applications today have AJAX calls that are made without proper error checking. This leads to applications with strangely empty select widgets, and requiring a hard reloads in orderto (hopefully) display properly. Network resiliance code is boilerplate, and it's often repeated for each call.

`hot-network` is used to wrap another element that makes AJAX calls, in order to make it

* Unclickable/gray while the AJAX call is going on
* (Optionally) manage errors in case so that it will:
  * Give a differently themed overlay if the call didn't work
  * Give the option to retry a failed AJAX call by clicking on the overlay
  * Communicate to the user in case of problems

The overlay is fully themable with CSS mixins.

# Basic use: make contained element unclickable while AJAX is going on

All you have to do in order to get a contained element unclickable while the AJAX call is going, is wrap it with `<hot-network>`. Make sure the widget making the AJAX call is the first child of `<hot-network>`:

    <hot-network>
      <form is="iron-form" id="iron-form" method="POST" action="/stores/polymer">
        <paper-input required id="name" name="name" label="Your name"></paper-input>
        <paper-button raised type="submit">Click!</paper-button>
      </form>
    </hot-network>

`<hot-network>` will only ever have two states: "loading" (widget is overlayed) or "loaded" (no AJAX call is happening).

# Usage with `<hot-form>`

The `<hot-form>` element is a mixed bag: it has itself a `request` property which is the one used to _initally load_ data from the server. This element is a "loader", and therefore should check for errors too (and reload the data if the loading call fails). However, `<hot-form>` _also_ always contains a `<form>` element that itself has a `request` property, used to submit data to the server.

In this case, you a recommended to use two `<hot-network>` elements:

* one to gray the form while loading initial data _and_ give users a chance to re-run the request in case of errors
* one to gray out the form while the data is being submitted (errors won't be managed since it wouldn't make sense here).

So:

    <hot-network manage-errors>
      <hot-form id="hot-form" record-id="57902ef29b880cd678a3d7a9">
        <hot-network>
          <form is="iron-form" id="iron-form" method="POST" action="/stores/polymer">
            <paper-input required id="name" name="name" label="Your name"></paper-input>
            <paper-input required id="surname" name="surname" label="Your surname"></paper-input>
            <paper-input required type="number" id="age" name="age" label="Your age"></paper-input>
            <paper-toggle-button id="active" name="active" label="Active?">Active?</paper-toggle-button>
            <paper-button raised type="submit">Click!</paper-button>
          </form>
        </hot-network>
      </hot-form>
    </hot-network>

Voila': you have a form that pre-loads and submits data, and that is 100% network-aware:

* During data-loading, the form will be gray
* If data-loading fails, users will be able to re-attempt it (and view an error message)
* While submitting data to the server, the form will be gray

Note that with just a small bunch of decorating widgets, you have here a fully functional, network-aware form.

# Error management

You can use `<hot-network>` for widgets that are build to display data after making an AJAX call. In this case, you will want to turn on `<hot-network>`'s error management:

    <hot-network manage-errors>
      <data-displayer></data-displayer>
    </hot-network>

In this case, `<hot-network>` will be in one of three states: "loading", "loaded", "error".

When in "error" state, `<data-displayer>` will still be overlayed and unreachable. However, when tapping on the overlay, the AJAX call will be attempted again (it will re-run the `generateRequest()` method of the `<iron-ajax>` element).

If `manage-error` is on, in case of error `<hot-network>` will fire a `user-message-error` event with the message in the detail, like this: `{ message: "Error!" }`. The error message can be customised by setting the `user-message-error` attribute:

    <hot-network manage-errors user-message-error="Cannot load users!">
      <data-displayer></data-displayer>
    </hot-network>

A UI widget should pick up this message event and display it to the user.

Note thjat `manage-errors` doesn't make sense in a form contest because 1) A form that has an error will likely emit its own user message event 2) A form that fails doesn't need to be overlayed to retry: the user will be able to click on the "Submit" button again.

# Targeting the right `<iron-ajax>`

`<hot-network>` should always directly wrap the element that makes the requests, which must be `<hot-network>`'s first child (also called the _target element_).
If the target element has a property called `request`, that will be assumed to be an `<iron-ajax>` widget. If it doesn't, then it's assumed that the target element itself is an `<iron-ajax>` element. The name of the property to be checked can be changed with the `target-property` attribute.
So:

    <hot-network>
      <!-The property 'request' of `<data-displayer>` is assumed to be an <iron-ajax ->
      <!-If 'request' is not there, then `<data-displayer>` itself is assumed to be an <iron-ajax ->
      <data-displayer></data-displayer>
    </hot-network>

    <hot-network target-property="iaw">
      <!-The property 'iaw' of `<data-displayer>` is assumed to be an <iron-ajax ->
      <!-If 'iaw' is not there, then `<data-displayer>` itself is assumed to be an <iron-ajax ->
      <data-displayer></data-displayer>
    </hot-network>

    <hot-network>
      <!-The <iron-ajax> element doesn't have a "request" attribute. ->
      <- So, <iron-ajax> will be the target ->
      <iron-ajax></iron-ajax>
    </hot-network>

So, the `request` property always has priority.

Although there haven't been any use cases where this has been necessary, it's also possible to specify the target ID directly and have the target further in the DOM, rather than being `<hot-network>`'s first child:

    <hot-network target-id="displayer">
      <div>
        <data-displayer></data-displayer>
      </div>
    </hot-network>

