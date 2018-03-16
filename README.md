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
