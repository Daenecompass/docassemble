---
layout: docs
title: Setting variables (and doing other things) with questions
short_title: Setting Variables
---

To instruct **docassemble** to store user input in a variable in
response to a [question], you need to include in your [`question`] a
variable name within a directive that indicates how you would like
**docassemble** to ask for the value of the variable.

# Directives

## <a name="yesno"></a><a name="noyes"></a>`yesno` or `noyes`

The `yesno` statement causes a question to set a boolean (true/false)
variable when answered.

{% include side-by-side.html demo="yesno" %}

In the example above, the web app will present "Yes" and "No" buttons
and will set `over_eighteen` to `True` if "Yes" is pressed, and
`False` if "No" is pressed.

The `noyes` statement is just like `yesno`, except that "Yes" means
`False` and "No" means `True`.

{% include side-by-side.html demo="noyes" %}

Note that yes/no fields can also be gathered on a screen along with
other fields; to make screens like that, use [`fields`] below.

## <a name="field with buttons"></a>`field` with `buttons`

A [`question`] block with a `buttons` statement will set the variable
identified in `field` to a particular value depending on which of the
buttons the user presses.

The `buttons` statement must always refer to a [YAML] list, so that
**docassemble** knows the order of the buttons.

If an item under `buttons` is a [YAML] key-value pair (written in the
form of `- key: value`), then the key will be the button label that the
user sees, and the value will be what the variable identified in `field`
will be set to if the user presses that button.

{% include side-by-side.html demo="buttons-labels" %}

An item under `buttons` can also be plain text; in that case
**docassemble** uses this text for both the label and the variable
value.

{% include side-by-side.html demo="buttons" %}

In other words, this:

{% highlight yaml %}
---
field: user_gender
question: What is your gender?
buttons:
  - Male
  - Female
---
{% endhighlight %}

is equivalent to this:

{% highlight yaml %}
---
field: user_gender
question: What is your gender?
buttons:
  - Male: Male
  - Female: Female
---
{% endhighlight %}

A powerful feature of `buttons` is the ability to use Python code to
generate button choices.  If an item under `buttons` is a key-value
pair in which the key is the word `code`, then **docassemble**
executes the value as Python code, which is expected to return a list.
This code is executed at the time the question is asked, and the code
can include variables from the interview.  **docassemble** will
process the resulting list and create additional buttons for each
item.

{% include side-by-side.html demo="buttons-code-list" %}

Note that the Python code needs to return key-value pairs (Python
dictionaries) where the key is what the variable should be set to and
the value is the button label.  This is different from the [YAML]
syntax.

This is equivalent to:

{% include side-by-side.html demo="buttons-code-list-equivalent" %}

You can use `buttons` as an alternative to [`yesno`] where you want
different text in the labels.

{% include side-by-side.html demo="yesno-custom" %}

## <a name="field with choices"></a>`field` with `choices`

To provide a multiple choice question with "radio buttons" and a
"Continue" button, use `field` with a `choices` list:

{% include side-by-side.html demo="choices" %}

## <a name="image button"></a>Adding images to `buttons` or `choices`

To add a decorative icon to a choice, use a key/value pair and add
`image` as an additional key.

{% include side-by-side.html demo="buttons-icons" %}

This works with `choices` as well:

{% include side-by-side.html demo="choices-icons" %}

## <a name="code button"></a>buttons/choices that embed [`question`] and [`code`] blocks

Multiple choice questions can embed [`question`] blocks and [`code`]
blocks.  These questions are just like ordinary questions, except they
can only be asked by way of the questions in which they are embedded.

You embed a question by providing a [YAML] key-value list (a
dictionary) (as opposed to text) as the value of a label in a
`buttons` or `choices` list.

{% include side-by-side.html demo="buttons-code-color" %}

While embedding [`question`] blocks can be useful sometimes, it is
generally not a good idea to structure interviews with a lot of
embedded questions.  You will have more flexibility if your questions
stand on their own.

It is also possible for multiple-choice questions to embed [`code`]
blocks that execute Python code.  (If you do not know what [`code`]
blocks are yet, read the section on [code blocks] first.)  This can be
useful when you want to set the values of multiple variables with one
button.

{% include side-by-side.html demo="buttons-code" %}

The question above tells **docassemble** that if the [interview logic]
calls for either `car_model` or `car_make`, the question should be
tried.  When the user clicks on one of the buttons, the code will be
executed and the variables will be set.

## <a name="field"></a>`field` without `buttons` or `choices`

{% include side-by-side.html demo="continue-participation" %}

A [`question`] with a `field` and no `buttons` will offer the user a
"Continue" button.  When the user presses "Continue," the variable
indicated by `field` will be set to `True`.

## <a name="signature"></a>`signature`

The `signature` directive presents a special screen in which the user
can sign his or her name with the trackpad or other pointing device.

{% include side-by-side.html demo="signature" %}

On the screen, the [`question`] text appears first, then the
[`subquestion`] text, then the signature area appears, and then the
`under` text appears.

In this example, the `user_signature` variable will be set to an
object of type [`DAFile`].  This variable can be included in the same
way that a document upload can be included.  For example:

{% highlight yaml %}
---
question: |
  Is this your signature?
subquestion: |
  ${ user_signature }
yesno: user_signature_verified
---
{% endhighlight %}

or, if you want to control the width of the image:

{% highlight yaml %}
---
question: |
  Is this your signature?
subquestion: |
  ${ user_signature.show(width='1in') }
yesno: user_signature_verified
---
{% endhighlight %}

## <a name="fields"></a>`fields`

The `fields` statement is used to present the user with a list of
fields.

{% include side-by-side.html demo="text-field-example" %}

The `fields` must consist of a list in which each list item consists
of one or more key/value pairs.  One of these keys (typically) is the
label the user sees, where the value associated with the key is the
name of the variable that will store the user-provided information for
that field.  The other key/value pairs in the item (if any) are
modifiers that allow you to customize how the field is displayed to
the user.

These modifiers are distinguished from label/variable pairs based on
the key; if the key is uses one of the names listed below, it will be
treated as a modifier; if it is anything else, it will be treated as a
label.

The following are the keys that have special meaning:

* `datatype`: affects how the data will be collected, validated and
  stored; see below.
* <a name="required"></a>`required`: the value can be `true` or
  `false`.  By default, all fields are required, so you never need to
  write `required: true` unless you want to.  Instead of writing
  `true` or `false`, you can write [Python] code.  This code will be
  evaluated for whether it turns out to be true or false.  For
  example, instead of `true` or `false`, you could use the name of a
  variable that is defined by a [`yesno`] question.
* <a name="hint"></a>`hint`: in text inputs, the value is provided as
  a [placeholder].  This can be used to suggest to the user what type
  of value should be entered.  Can contain [Mako] text.
* <a name="help"></a>`help`: the value is explanatory help text that
  appears when user clicks on the label.  The label will be green to
  indicate that it can be clicked on.  Can contain [Mako] text.
* <a name="default"></a>`default`: the value will be set as the
  default value of the field.  Can contain [Mako] text.
* <a name="choices"></a>`choices`: a list of possible options for a
  multiple choice field.  Can be a list of key/value pairs (key is
  what the variable will be set to; value is the label seen by the
  user) or a list of plain text items (in which case the label and the
  variable value are the same).  When the `datatype` is `object`,
  `object_radio`, or `object_checkboxes`, [`choices`](#choices)
  indicates list of objects from which the user will choose.
* `exclude`: this is used in combination with [`choices`](#choices) when the
  `datatype` is `object`, `object_radio`, or `object_checkboxes`.  Any
  object in `exclude` will be omitted from the list of choices if it
  is present in [`choices`](#choices).
* [`code`]: code that generates a list of possible options for a
  multiple choice field.
* `shuffle`: used with [`code`] or [`choices`](#choices), when `true` it
  randomizes the order of the list of choices; the default is not to
  "shuffle" the list.
* <a name="show if"></a>`show if`: you can use this if you want the
  field to be shown only if another field on the same screen is set to
  a particular value.  It must refer to a [YAML] dictionary with two keys:
  `variable` and `is`, where `variable` refers to the variable name of
  the other field, and `is` refers to the value that will cause the
  field to be shown.
* <a name="hide if"></a>`hide if`: this works just like `show if`,
  except that it hides the field instead of showing it.
* `disable others`: if set to `true`, then when the user changes the
  value of the field to something, all the other fields in the
  question will be disabled.
* <a name="note"></a>`note`: the value is [Markdown] text that will
  appear on the screen; useful for providing guidance to the user on
  how to enter information.
* `label`: Instead of expressing your labels and variable names in the
  form of `- Label: variable_name`, you can specify a label using the
  `label` key and the variable name using the `field` key.
* `field`: See the explanation of `label` above.
* `html`: like `note`, except raw HTML.
* `script`: raw HTML to be appended to the bottom of the page; usually
  used for Javascript code that interacts with HTML specified in
  `html` entries.
* `css`: raw HTML to be appended to the HTML head; usually used to
  provide css classes for HTML specified in `html` entries.
* <a name="no label"></a>`no label`: if you use `no label` as the
  label for your variable, the label will be omitted.  On wide
  screens, the field will fill more of the width of the screen if the
  label is set to `no label`.  To keep the width of the field normal,
  but have a blank label, use `""` as the label.

The possible `datatype` values are:

* `text`: a single-line text input box (the default).
* <a name="area"></a>`area`: a multi-line text area
* <a name="date"></a>`date`: a valid date.
* <a name="email"></a>`email`: a valid e-mail address.
* <a name="integer"></a>`integer`: a valid whole number.
* <a name="number"></a>`number`: a valid numeric value.
* <a name="currency"></a>`currency`: a valid numeric value; input box
  shows a currency symbol based on locale defined in the
  [configuration].
* <a name="file"></a>`file`: a single file upload (a [`DAFileList`] object results).
* <a name="files">`files`: single or multiple file upload (a [`DAFileList`] object
  results).
* <a name="camera"></a>`camera`: like `file`, except with HTML5 that
  suggests using the device's camera to take a picture;
* <a name="camcorder"></a>`camcorder`: like `camera`, except for
  recording a video;
* <a name="microphone"></a>`microphone`: like `camera`, except for
  recording an audio clip;
* `yesno`: checkbox with label, aligned with labeled fields.
* `yesnowide`: checkbox with label, full width
  of area.
* `checkboxes`: show the [`choices`](#choices) list as checkboxes;
  variable will be a dictionary with items set to true or false
  depending on whether the option was checked.  No validation is done
  to see if the user selected at least one, regardless of the value of
  `required`.
* `object`: this is used when you would like to use a variable to
  refer to an existing object.  You can provide the list of objects to
  choose from in one of two ways.  The first way is to list the object
  names under `selections`.  The second way is to provide [`code`]
  that calls the `selections()` [function].  The user will be
  presented with a pull-down selector.
* `object_radio`: like `object`, except the user interface uses radio
  buttons rather than a pull-down list.  See [groups] for more information.
* `object_checkboxes`: this is used when you would like to use a
  question to set the elements of an object of type [`DAList`].  The
  choices in [`choices`](#choices) (optionally modified by `exclude`)
  will be presented to the user as checkboxes.  The `.gathered`
  attribute of the variable will be set to `True` after the elements
  are set.  See [groups] for more information.
* `radio`: show a [`choices`](#choices) list as a list of radio
  buttons instead of as a dropdown [select] tag (which is the
  default).  The variable will be set to the value of the choice.

For some field types, you can require input validation by adding the
following to the definition of a field:

* `min`: for `currency` and `number` data types, require a minimum
  value.
  [See here for more information](http://jqueryvalidation.org/min-method).
* `max`: for `currency` and `number` data types, require a maximum
  value.
  [See here for more information](http://jqueryvalidation.org/max-method).
* `minlength`: require a minimum number of characters in a textbox,
  number of checkboxes checked, etc.
  [See here for more information](http://jqueryvalidation.org/minlength-method).
* `maxlength`: require a maximum number of characters in a textbox,
  number of checkboxes checked, etc.
  [See here for more information](http://jqueryvalidation.org/maxlength-method).

Here is a long example that illustrates many of these features.

{% include side-by-side.html demo="fields" %}

The referenced [CSS file] contains the following:

{% highlight css %}
.mytime {
    color: green;
}
{% endhighlight %}

## <a name="mc-code"></a>Multiple-choice questions in `fields` with choices from code

Note that adding [`code`] to a field makes it a multiple-choice
question.  If you have a multiple-choice question and you want to
reuse the same selections several times, you do not need to type in
the whole list every time.  You can define a variable to contain the
list and a [`code`] block that defines the variable.  For example:

{% include side-by-side.html demo="fields-mc" %}

## Assigning existing objects to variables

Using [Mako] template expressions ([Python] code enclosed in `${ }`), you can
present users with multiple-choice questions for which choices are
based on information gathered from the user.  For example:

{% highlight yaml %}
---
include: basic-questions.yml
---
question: |
  What is your favorite date?
fields:
  - Greatest Date Ever: favorite_date
    datatype: date
    choices:
      - ${ client.birthdate }
      - ${ advocate.birthdate }
---
question: |
  The best day in the history of the world was ${ favorite_date }.
sets: all_done
---
mandatory: true
code: all_done
---
{% endhighlight %}

But what if you wanted to use a variable to refer to an object, such
as a person?  You could try something like this:

{% highlight yaml %}
---
question: |
  Who is the tallest?
fields:
  - Tallest person: tallest_person
    choices:
      - ${ client }
      - ${ advocate }
---
{% endhighlight %}

In this case, `tallest_person` would be set to the _name_ of the
`client` or the _name_ of the `advocate`.  But what if you wanted to
then look at the birthdate of the tallest person, or some other
attribute of the person?  If all you had was the person's name, you
would not be able to do that.  Instead, you would want
`tallest_person` to be defined as the object `client` or the object
`advocate`, so that you can refer to `tallest_person.birthdate` just
as you would refer to `client.birthdate`.

You can accomplish this by setting `datatype` to `object` within a
`fields` list, where the [`choices`](#choices) are the names of the objects from
which to choose.  (Optionally, you can set a `default` value, which is
also the name of a variable.)

For example:

{% highlight yaml %}
---
include: basic-questions.yml
---
question: Who is the villain?
fields:
  no label: villain
  datatype: object
  default: client
  choices:
    - client
    - advocate
---
question: |
  The villain, ${ villain }, was born on
  ${ format_date(villain.birthdate) }.
sets: all_done
---
mandatory: true
code: all_done
---
{% endhighlight %}

([Try it out here]({{ site.demourl }}?i=docassemble.demo:data/questions/testobjectfield.yml){:target="_blank"}.)

Note that this interview incorporates the [`basic-questions.yml`] file
which defines objects that are commonly used in [legal applications],
including `client` and `advocate`.  It also contains questions for
asking for the names of these people.

The interview above presents the names of the `client` and the
`advocate` and asks which of these people is the villain.

If the user clicks the name of the advocate, then **docassemble** will
define the variable `villain` and set it equal to `advocate`.

Note that because `advocate` is an [object], `villain` will be an
_alias_ for `advocate`, not a _copy_ of `advocate`.  If you
subsequently set `advocate.birthdate`, you will immediately be able
retrieve that value by looking at `villain.birthdate`, and vice-versa.

Also because `villain` is an alias, if you refer to
`villain.favorite_food` and it is not yet defined, **docassemble**
will go searching for a question that offers to define
`advocate.favorite_food`.  This is because **docassemble** objects
have an intrinsic identity, a unique name given to them at the time
they are created.  (You can inspect this by referring to
`villain.instanceName` in a question and will see that it returns
`advocate`.)  For more information about this, see the discussion in
the documenation for [DAObject].  (All **docassemble** objects are
subtypes of [DAObject].)

If any of the objects listed under `choices` represent lists of
objects, such as `case.defendant` or `client.child` (objects of type
`PartyList`, those lists will be expanded and every item will be
included.  You can also include under `choices` [Python] code, such as
`case.parties()` or `case.all_known_people()`.

The `datatype` of `object` presents the list of choices as a
pull-down.  If you prefer to present the user with radio buttons, set
the `datatype` to `object_radio`.

## <a name="sets"></a>`sets`

A `sets` line tells **docassemble** that if it is looking for the
value of a particular variable, it should try asking the question in
the question block containing the `sets` statement.

{% include side-by-side.html demo="sets" %}

This does not necessarily mean that the variable identified in `sets`
will actually be set.  For example, the question above does not allow
the user to do anything.

When used as an "empty promise" of setting a variable, the `sets`
statement is useful for creating the final screen that the user sees
before exiting the interview.  For example, the [interview logic] can
cause the "question" above to appear by including a final line of code
such as:

{% highlight text %}
need(user_done)
{% endhighlight %}

The `user_done` variable will never be set to any value, but that is
ok, because you don't want the user to be able to "get past" the final
screen.  See [functions] for an explanation of [`need()`].

## <a name="sets special"></a>`sets` with special buttons/choices 

The `sets` statement is also used in conjunction with `buttons` in
multiple choice questions where there is no `field` to be set.

{% include side-by-side.html demo="sets-exit" %}

The above example allows the user to "exit" the interview (be redirected
to a specific web site that is pre-set in the **docassemble**
[configuration] or "restart" the interview (go back to the beginning).

There are six special button functions:

* `restart`
* `exit`
* `leave`
* `continue`
* `refresh`
* `signin`

`restart` resets the user's variable store, except that any parameters
that were originally passed through as URL parameters will be used again.

`exit` means that the user's variable store will be reset and the user
will be redirected either to the URL given by the associated `url`
text, or if no `url` is defined, to the `exit` page defined in the
[configuration].  If the user tries to come back to the interview
again, he will start the interview again, as though it had never been
started.  Original URL parameters will be lost.

For example:

{% include side-by-side.html demo="sets-exit-url" %}

[Mako] can be used in the `url` text.

`leave` works like `exit` except that the user's variable store will be left
intact.  This means that if the user comes back to the interview
again, he will pick up where he left off.

`continue` means that **docassemble** will look for another question
in the interview that might define the necessary variable.

`refresh` re-runs the [interview logic].  It has much the same effect as
refreshing the page in the browser.  It is useful in multi-user
interviews when the user is waiting for another user to finish
entering information.  It can also be useful in interviews that use
external data sources.

`signin` redirects the user to the **docassemble** sign-in page.

Instead of using `buttons`, you can use `choices` to get a radio list
instead of a selection of buttons.

{% include side-by-side.html demo="sets-exit-choices" %}

The functionality is the same.

## <a name="event"></a>`event`

The `event` line acts just like `sets`: it advertises to
**docassemble** that the question will potentially define a variable.

{% include side-by-side.html demo="event-role-event" %}

In the example above, the `event` line tells **docassemble** that this
[`question`] should be displayed to the user if **docassemble**
encounters the `role_event`, which is a special "event" that can
happen in multi-user interviews (see
[roles]({{site.baseurl}}/docs/roles.html)).  The event is triggered
when the interview reaches a point when a person other than the
current user needs to answer a question.  For example, while a client
is filling out an interview, the [interview logic] might call for a
variable that can only be set by an advocate who reviews the client's
answers.  In this scenario, a `role_event` will be triggered.  When
this happens, **docassemble** will look for a [`question`] or [`code`]
block that defines the variable `role_event`, and it will find the
example question above.

This directive can also be used to create screens that the user can
reach from the menu or from hyperlinks embedded in question text.  For
more information, see [url_action()], [process_action()],
[action_menu_item()], and [menu_items].

## <a name="review"></a>review

A `review` block allows interview authors to provide a screen where
users can review and edit their answers.  Typically, the user will get
to this screen by selecting an option from the web app menu (e.g.,
"Review Answers"), or by clicking on a hyperlink within `subquestion`
text (e.g., "to review the answers you have provided so far, click
here").

Here is an example of a `review` block that is launched from the menu:

{% highlight yaml %}
---
event: review_answers
question: |
  Revisit questions
subquestion: |
  These are the questions you have answered so far.  Click to revisit.
review:
  - Favorite fruit: fruit
  - Favorite vegetable: vegetable
  - Favorite fungus: fungi
---
mandatory: true
code: |
  menu_items = [ action_menu_item('Review Answers', 'review_answers') ]
---
{% endhighlight %}

If `fruit` and `vegetable` have been defined, and the user selects
"Review Answers" from the menu, he or see will see this:

![Screenshot of review]({{ site.baseurl }}/img/review-block.png)

([Try it out here]({{ site.demourl }}?i=docassemble.demo:data/questions/testreview.yml){:target="_blank"}.)

Note that the `review` block does not show a link for "Favorite
fungus" because the variable `fungi` has not been defined yet.
However, once `fungi` is defined, the `review` block would show it.

This behavior is different from the typical behavior of
**docassemble** blocks.  Normally, referring to a variable that has
not yet been defined will trigger the asking of a question that will
define that variable.  In the `review` block, however, the presence of
an undefined variable simply causes the item to be omitted from the
display.

For more information about adding menu items, see the sections on
[special variables] and [functions].

### <a name="review customization"></a>Customizing the display of `review` options

You can provide the user with a review of answers and buttons that the
user can press to revisit an answer:

{% highlight yaml %}
---
event: review_answers
question: |
  Revisit your answers
review:
  - Revisit fruit: fruit
    button: |
      You said your favorite fruit was ${ fruit }.
  - Revisit vegetable: vegetable
    button: |
      You said your favorite vegetable was ${ vegetable }.
  - Revisit fungus: fungi
    button: |
      You said your favorite fungus was ${ fungi }.
---
{% endhighlight %}

This results in a screen that looks like this:

![Screenshot of review with buttons]({{ site.baseurl }}/img/review-block-buttons.png)

([Try it out here]({{ site.demourl }}?i=docassemble.demo:data/questions/testreview2.yml){:target="_blank"}.)

In addition, the `review` block, like the `fields` block, allows you
to use `note`, `html`, `script`, and `css` entries.

If these are modified with the optional `show if` modifier, they will
only be displayed if the variable referenced by the `show if` modifier
has been defined.  In addition, if any of these entries refer to a
variable that has not been defined yet, they will be omitted.

{% highlight yaml %}
---
event: review_answers
question: |
  Review your answers
review:
  - note: |
      Revisit your food preferences.
    show if: fruit
  - Favorite fruit: fruit
  - Favorite vegetable: vegetable
  - Favorite fungus: fungi
---
{% endhighlight %}

![Screenshot of review with note]({{ site.baseurl }}/img/review-block-note.png)

([Try it out here]({{ site.demourl }}?i=docassemble.demo:data/questions/testreview3.yml){:target="_blank"}.)

The `review` block allows you to add `help` text to an entry, in
which case the text is shown underneath the hyperlink.  If this text
expects a variable to be defined that has not actually been defined,
the item will not be shown.  Note: this is not available with the
`button` display format.

{% highlight yaml %}
---
event: review_answers
question: |
  Review your answers
review:
  - Favorite fruit: fruit
    help: |
      You indicated you liked ${ fruit }.
  - Favorite vegetable: vegetable
    help: |
      You indicated you liked ${ vegetable }.
  - Favorite fungus: fungi
    help: |
      You indicated you liked ${ fungi }.
---
{% endhighlight %}

![Screenshot of review with help]({{ site.baseurl }}/img/review-block-help.png)

([Try it out here]({{ site.demourl }}?i=docassemble.demo:data/questions/testreview4.yml){:target="_blank"}.)

### <a name="review auto"></a>Why can't `review` blocks be automatically generated?

The list of variables to display to the user in a `review` block needs
to be specified by the interview author.  There are several reasons
why this needs to be done manually as opposed to automatically:

1. Variables in your interview may be interdependent.  You do not
   necessarily want to allow the interviewee to edit any past answer
   at will because this may result in internal inconsistencies or
   violations of the logic of your interview.  For example, if your
   interview has a variable called `eligible_for_medicare`, which is
   set after the user answers a series of questions, you would not
   want the user to be able to go back and set his or her age to 30,
   at least not without a reconsideration of the definition of
   `eligible_for_medicare`.  Therefore, it is important that the
   interview author control what the user can edit.
2. A list of answers already provided might not be user-friendly
   unless the interview author presents it in a logically organized
   fashion.  The order in which the questions were asked is not
   necessarily the most logical way to present the information for
   editing.

# <a name="variable names"></a>A note about variable names

Variable names are [Python identifiers], which means they can be any
sequence of uppercase or lowercase letters, digits, and underscores,
except the first character cannot be a digit.  No spaces are allowed
and no punctuation is allowed except for the underscore, `_`.

The following are valid variable names:

* `fried_fish1`
* `NyanCat`
* `nyancat` (variables are case-sensitive, so this is not the same as
  the above)
* `__f645456DG_greij_43` (but why would you use something so ugly?)
* `USER_PHONE_NUMBER` (ok, but why are you yelling?)

The following are **not** valid variable names, and if you try to use
such variable names you will may get an error or unexpected results:

* `8th_plaintiff` (you can't begin a variable name with a number;
  [Python] will say "invalid syntax")
* `fried.fish1` (this is valid code, but [Python] will think you are
referring to the attribute `fish1` of the object `fried`)
* `user's_phone_number` (apostrophes are not allowed; [Python]
  recognizes them as single quotes)
* `favorite animal` (spaces are not allowed)
* `beneficiary#1` (punctuation marks other than `_` are not allowed)
* `applicant_résumé` (only plain alphabet characters can be used)

See [reserved variable names] for a list of variable names that you
cannot use because they conflict with built-in names that [Python] and
**docassemble** use.

[configuration]: {{ site.baseurl }}/docs/configuration.html
[select]: http://www.w3schools.com/tags/tag_select.asp
[placeholder]: http://www.w3schools.com/tags/att_input_placeholder.asp
[code blocks]: {{ site.baseurl }}/docs/code.html
[Mako]: http://www.makotemplates.org/
[Markdown]: https://daringfireball.net/projects/markdown/
[YAML]: https://en.wikipedia.org/wiki/YAML
[object]: {{ site.baseurl }}/docs/objects.html
[objects]: {{ site.baseurl }}/docs/objects.html
[Python identifiers]: https://docs.python.org/2/reference/lexical_analysis.html#identifiers
[reserved variable names]: {{ site.baseurl }}/docs/reserved.html
[Python]: https://en.wikipedia.org/wiki/Python_%28programming_language%29
[question]: {{ site.baseurl }}/docs/questions.html
[CSS file]: {{ site.demourl }}/packagestatic/docassemble.demo/my.css
[function]: {{ site.baseurl }}/docs/functions.html
[functions]: {{ site.baseurl }}/docs/functions.html
[special variables]: {{ site.baseurl }}/docs/special.html
[legal applications]: {{ site.baseurl }}/docs/legal.html
[interview logic]: {{ site.baseurl }}/docs/logic.html
[DAObject]: {{ site.baseurl }}/docs/objects.html#DAObject
[url_action()]: {{ site.baseurl }}/docs/functions.html#url_action
[process_action()]: {{ site.baseurl }}/docs/functions.html#process_action
[action_menu_item()]: {{ site.baseurl }}/docs/functions.html#action_menu_item
[menu_items]: {{ site.baseurl }}/docs/special.html#menu_items
[`question`]: {{ site.baseurl }}/docs/questions.html#question
[`subquestion`]: {{ site.baseurl }}/docs/questions.html#subquestion
[`code`]: {{ site.baseurl }}/docs/code.html
[`DAFile`]: {{ site.baseurl }}/docs/objects.html#DAFile
[`DAFileList`]: {{ site.baseurl }}/docs/objects.html#DAFileList
[`DAList`]: {{ site.baseurl }}/docs/objects.html#DAList
[`need()`]: {{ site.baseurl }}/docs/functions.html#need
[`basic-questions.yml`]: {{ site.github.repository_url }}/blob/master/docassemble_base/docassemble/base/data/questions/basic-questions.yml
[`yesno`]: #yesno
[groups]: {{ site.baseurl }}/docs/groups.html