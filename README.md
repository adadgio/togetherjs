walkabout.js
============

An automatic jQuery app tester.

This code figures out what your application is paying attention to,
and does it.  It fills in fields with random values.  It finds the
events you listen for and fires those events.  It finds internal links
(like `<a href="#foo">`) and clicks them.  It's a little like a fuzz
tester for your app.

You can use it like so:

```javascript
// This makes something like $('#some-input').val() return random values:
jQuery.fn.val.patch();

// Now, fiddle around, do a 100 random things:
$(document).runManyActions(100);
```

You can hint about what's a valid input for something:

You can add `data-walkabout-disable="1"` to any element to suppress
activation of that element.

You can use `data-walkabout-eventname="..."` to set attributes on the
event that is created, such as `data-walkabout-keyup="{which: 13}"`

You can use `data-mock-options="['a', 'b']"` to give the valid inputs
for a field.

Non-jQuery Support
------------------

There's some experimental support for working with code that doesn't
use jQuery.  It *might* work with other frameworks, but is more
intended to work with code that doesn't use a framework.

This technique uses code rewriting to capture calls to
`.addEventListener()` and `.value`.  The code can't use any tricks, it
needs to actually use those literal names -- which is usually fine in
"normal" code, but might not be in fancy or optimized code.  E.g.,
`el["addEventListener"](...)` would not be found.

You can use `Walkabout.rewriteListeners(code)` to rewrite the code.

The code transformation looks like this:

```javascript
$("#el")[0].addEventListener("click", function () {}, true);
var value = $("#textarea")[0].value;

// Becomes:
Walkabout.addEventListener($("#el")[0], "click", function () {}, true);
var value = Walkabout.value($("#textarea")[0]);
```

And to find actions you use `Walkabout.findActions(element)`.  There
is currently no equivalent to `$.runManyActions()`.

To Do
-----

Lots of stuff, of course.  But:

- Sometimes the validation options (like `data-mock-options`) should
  be obeyed, and sometimes they should be ignored.  Obeying them
  progresses you through the site, disobeying them does some fuzz
  testing.

- Everything is terribly duplicated between the jQuery and non-jQuery
  support.  Needs some refactoring.

- Not all form controls get triggered, I think.  E.g., checkboxes
  don't get checked.

- `.val()` should figure out better what's a reasonable return value.
  E.g., a textarea returns multi-line strings, a checkbox returns
  true/false.

- The generator could have smarts about what scripts are successful
  and which are not.  Starting with a successful script (that
  progresses you through the application), and then tweaking that
  script and increasing randomness at the end of the script.

- We could look at code coverage as a score.  Or we could even just
  look at event coverage - hidden elements often have events bound,
  but we know they have to be visible to be triggered.

- Something more Gaussian with the values generated.  Like, sometimes
  you should make a 100 character input.  But maybe not as often as a
  10 character input.  And sometimes a 1 character input.
