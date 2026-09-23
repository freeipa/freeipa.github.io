WebUI_keyboard_confirmation
===========================



Keyboard confirmation of dialogs
================================

Purpose
-------

Make a common and a better way of confirming/cancelling of dialogs by
keyboard. Original implementation allowed only limited support which
wasn't very user-friedly. Purpose of this enhancement is to have default
positive (confirmation) and negative (cancellation) actions which would
be executed on corresponding (*enter*/*escape*) key press. It should
make some common WebUI use-cases quicker.



Feature design
--------------

Confirmation
----------------------------------------------------------------------------------------------

Confirmation is the dialog's default positive action. It corresponds to
the purpose of the dialog ie. 'Add user', 'Disable user', 'Remove DNS
zone permission'. If dialog has multiple positive actions, like in
entity adder dialog, the default action should match the first button's
action.

Confirmation is initiated on 'enter' key press anywhere on the dialog's
space except in cases when *enter* serves for some widget's action,
these are:

-  links - used for action (add new value) or navigation
-  buttons - allow to execute other action like *Add and add another*
-  textareas - new line
-  selects - opens a list of options

Cancellation
----------------------------------------------------------------------------------------------

Cancellation is default negative action. It corresponds to a negation of
dialog's purpose. Usually it is the action associated with the last
button of the dialog.

Cancellation is initiated by 'escape' key. Same rules apply as in
confirmation with the exception of known exceptions (there are none yet
as there are none conflicts with widgets).



Dialog stacking
---------------

Multiple dialogs can be opened at the same time. We have to make sure
that when the top one is closed the current most recently opened dialog
should be the top one. It should also gain focus (preferably on the
first editable element) to allow keyboard usage.



Original implementation
-----------------------

User had to use *tab* key several times to focus a button and then hit
*enter* to tell the browser to push the button. This method could be
used for both confirming and canceling. Hitting the 'escape' key closed
the dialog which can be considered as cancellation for most cases.