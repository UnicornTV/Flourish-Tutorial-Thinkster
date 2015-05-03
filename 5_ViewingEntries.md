# Viewing our Saved Entries

## Introduction
Alright now that we've successfully saved our journal entry, we need to a be able
to view our saved entries. In this chapter we are going to set up our Journal 
view, where we'll see a list of our entries. 

New Concepts This Chapter 
* UITableView
* Responder Chain


## A Smooth Transition

We'll start off by designing a transition from our Entry view to our soon-to-exist
journal view. What do I mean by designing a transition? A well-designed app guides
a user through an experience. One of the biggest considerations for any experience
is navigation. When do you take a user somewhere and when do you allow the user
to freely navigate? Those choices severely impact your UX and can make your app
feel better or worse to the user. Going through our choices in Flourish will show
you illustrate some of these considerations. We're going to transition to a new 
view, but for now we need to clean up our UX in EntryFormController.swift.

We left the last chapter by simply logging a
success message, but we haven't shown the user anything. As a rule of thumb, 
always show the user when an action he/she has taken fails or succeeds. This all
relates to the larger concept of user feedback: signaling to the user that 
an action has consequences by altering the UI somehow. Let's start of with a 
simple example.

## Dismissing Keyboard 

Let's dismiss the keyboard when a user taps outside an input element. We'll
do this by using the touchesBegan:withEvent method of the UIResponderClass. 

{x: dismiss_entry_keyboard}
In EntryFormController.swift, add the following code:

~~~language-swift
override func touchesBegan(touches: Set<NSObject>, withEvent event: UIEvent) {
        view.endEditing(true)
    }
~~~~

UIResponder is a class that respond to and handle touch and motion events. 
touchesBegan:withEvent is fired when one or more fingers touch down in a view to
start a multitouch sequence. A multitouch sequence contains four phases. 
During each phase touches on the screen, the app sends a series of event messages 
to the responder and the responder can handle the messages by implementing 
UIResponder class methods.

The first of these event messages is handled by the 
touchesBegan class we're going to implement. The first parameter is a set of 
touches that are new or have changed in this phase of the touch sequence. This
set of type Set. For those that missed, Swift now has a native Set class so no 
more NSSet in these UIResponder class methods. The second paramter is a UIEvent
class object containing all of the UITouch objects in the <i>entire</i> sequence, 
not just in this phase.

Inside touchesBegan:withEvent we could do all sorts of interesting things, but 
all we care about is dismissing our keyboard, which can be accomplished by using
the endEditing() method of UIView. When we tap a text field or some other input, 
it becomes the first responder. That means any touch events are handled by that
input field before anything else. When an input element is a first responder, our
keyboard shows up. By passing YES into this method, we force our text field to 
resign first responder status, which hides the keyboard. 

Finally, be sure to use the override prefix before you declare your function 
because there is always a default implementation of touchesBegan:withEvent that 
doesn't do anything, so we need to override that with our method. 

{x: check_keyboard_UX}
Build and run to make sure the keyboard is dismissed when you tap outside of
an input element. 

## Showing Success/Failure Alerts 

Our next UX improvement is to show success failure alert messages to the user. 
Before iOS 8, alerts were of class UIAlertView, but as of iOS 8.0 we have the 
simpler UIAlertController class that we can use to show alerts. UIAlertView
also has an addAction() method that we use to configure buttons in the alert
and tie those buttons to actions. 

{x: success_alert}
In the success completion block of our saveForm() method, add present an alert
with the following code: 

~~~language-swift
let statusAlert = UIAlertController(title:"Entry Saved", message: "Your entry has been saved!", preferredStyle: .Alert)
statusAlert.addAction(UIAlertAction(title:"OK", style: .Default, handler: nil))
self.presentViewController(statusAlert, animated: true, completion: nil)
~~~~

The first line of our code declares a constant called statusAlert and we set it
equal to an instance of the UIAlertController class. Our instance is initialized
with three parameters: title, message, and preferred style. Title is a string that
will be the heading in the alert. Message is the body copy of the alert, while 
prefferedStyle is a really a choice between alert and action. In this case we
want our alert to look like, well, an alert. 

The second line of our code adds an action button to our alert by using the addAction() 
method of UIAlertController. In that method we pass in a title, which sets the 
button's label, a style, which we just set to default, and a completion handler, 
which we don't need right now. By default pressing an alert button will dismiss
the alert, and that's all we need for our success alert. 

The third line of our code uses the presentViewController method of UIViewController
to add our alert to our view hierarchy. We set the animated parameter to true and
we don't need a completion block. 

Now we need a similar alert for when our saving fails. 

{x: error_alert}
In the error completion block of our saveForm() method, add present an alert
with the following code: 

~~~language-swift
let statusAlert = UIAlertController(title:"Entry Saved", message: "Your entry has been saved!", preferredStyle: .Alert)
statusAlert.addAction(UIAlertAction(title:"OK", style: .Default, handler: nil))
self.presentViewController(statusAlert, animated: true, completion: nil)
~~~~

