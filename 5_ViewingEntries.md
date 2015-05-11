# Viewing our Saved Entries

## Introduction
Alright now that we've successfully saved our journal entry, we need to a be able
to view our saved entries. In this chapter we are going to set up our Journal 
view, where we'll see a list of our entries. 

New Concepts This Chapter 
* UITableView
* Responder Chain


## Improving Entry Form UX

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
~~~

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
~~~

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

{x: test_success_modal}
Build and run and save an entry. Your alert should look like the image below.

![success_modal](/tuts_images/success_modal.png)

Now we need a similar alert for when our saving fails. 

{x: error_alert}
In the error completion block of our saveForm() method, add present an alert
with the following code: 

~~~language-swift
let statusAlert = UIAlertController(title:"Error", message: error!.localizedDescription, preferredStyle: .Alert)
statusAlert.addAction(UIAlertAction(title:"OK", style: .Default, handler: nil))
self.presentViewController(statusAlert, animated: true, completion: nil);
~~~

The only difference between our success and error alerts is that we don't want 
to hardcode a message, rather we want to pass the error message we return into 
the message parameter of the UIAlertController init method. Our error is an object
of type NSError, yet the message parameter needs to be a string. Luckily, NSError
has a handy property called localizedDescription, which is a string containing 
the localized description of the error. So all we need to do is unwrap the error
optional and pass the localizedDescription property as our message parameter. 


{x: test_success_modal}
Build and run to test your error alert. The easiest way to get an error thrown 
is to log out of iCloud in the emulator and try to save an entry. Your alert view
should look like the image below

![error_modal](/tuts_images/error_modal.png)

## Failure Handler

Now that we've given our user some success/error feedback in the form of alert
views, we can finish up by giving the user an option to retry the save from the 
alert view. When building an app for mobile, retry is an important function because
wifi/4G/LTE signals can be unreliable. It could be that a user simply lost service
for a split second as they went to submit the entry, so we want to make it as 
easy as possible to attempt it again. 

We already tied our saveForm method to the touchUpInside event of our submit 
button, so all we need to do is programmatically call the touchUpInside event of
the submit button from our retry method. First step is accessing the submitButton
as an IBOutlet. 

{x: create_submit_iboutlet}
Write the following below your other variable and constant declarations: 

~~~language-swift
  @IBOutlet weak var submitButton: UIButton!
~~~ 

{x: connect_sumbit_to_storyboard} 
With both our main.storyboard file and our EntryFormController.swift files open, 
Ctrl-click on the save entry button in the storyboard and drag it onto the submitButton
variable declaration. 

{x: create_reset_form}
Create a retrySave() function with the following code:

~~~language-swift
  func retrySaving(alert: UIAlertAction!) {
        submitButton.sendActionsForControlEvents(UIControlEvents.TouchUpInside)
    }
~~~ 

Our retrySaving function takes an alert parameter of type UIAlertAction. UIAlertAction
is simply an object representing an action that can be taken when tapping a 
button in an alert. Since we are going to be calling this method from an alert, 
we need this parameter. Inside the function we are using the sendActionsForControlEvents()
method of UIControl to fire the touchUpInside event on our submitButton. This 
will then call whatever IBAction is assigned to the touchUpInside event of our
submitButton, which is our saveForm function. 

{x: add_reset_action}
Now that we have our function and IBOutlet, we need to add an action button to 
our alert view that calls our retrySaving() function it is handler parameter:

~~~language-swift
statusAlert.addAction(UIAlertAction(title:"Retry", style: .Default, handler: self.retrySaving))
~~~ 

Your saveForm method should now look like this:

~~~language-swift
@IBAction func saveForm(sender: AnyObject) {
    let entry = Entry(title:titleInput.text, body:bodyInput.text, mood: feelingButton.tag, location: currentLocation)
    
    entry.create() {
        success, message, error in
        if success {
            println("success!", message)
            self.titleInput = nil
            self.feelingButton.setTitle("select", forState: .Normal)
            self.feelingButton.setTitleColor(UIColor(rgba:"#3687FF"), forState: .Normal)
            self.bodyInput.text = nil
            let statusAlert = UIAlertController(title:"Entry Saved", message: "Your entry has been saved!", preferredStyle: .Alert)
            statusAlert.addAction(UIAlertAction(title:"OK", style: .Default, handler: nil))
            self.presentViewController(statusAlert, animated: true, completion: nil);
        }
        else {
            let statusAlert = UIAlertController(title:"Error", message: error!.localizedDescription, preferredStyle: .Alert)
            statusAlert.addAction(UIAlertAction(title:"OK", style: .Default, handler: nil))
            statusAlert.addAction(UIAlertAction(title:"Retry", style: .Default, handler: self.retrySaving))
            self.presentViewController(statusAlert, animated: true, completion: nil)
        }
    }
}
~~~ 

## Adding Journal View

Now that we've cleaned up the UX in our Entry view, we can turn our attention to
our Journal view, which will display our saved entries. 

{x: new_swift_file}
Add a new swift file to your project. Go to file > new > file and select 
"swift file." Name the file JournalController.swift

{x: new_class_declaration}
Erase any code in the JournalController.swift file and replace it with the
following:

~~~language-swift
import UIKit

class JournalController: UITableViewController
{
    
}
~~~

Our class declaration should be familiar to you by now, with the only difference
being our JournalController class of type UITableViewController, not 
UIViewController. UITableViewController inherits from UIViewController and as 
you can probably guess, creates a controller object that manages a table view. 
Our Journal view is essentially a table with each cell representing a saved entry. 

We currently have a view controller associated with the journal tab. We need to 
replace that with a UITableViewController.

{x: remove_second_view_controller}
Go to main.storyboard and delete the view controller associated with the journal 
tab item. 

{x: add_table_to_tab} 
Drag a table view controller object from the object library and drop it onto the 
storyboard and ctrl-drag an outlet from the navigation controller scene to the 
new view scene. When you drop the outlet, select "root view controller" from the 
Relationship Segue menu. 

![link_table_view](/tuts_images/table_view_controller_tab_link.png)

{x: linking_journal_file_to_storyboard}
Go to main.storyboard and click on the table view controller object we just added to the Journal scene. Now go to that view controller's identity inspector and in the Custom Class section's class field, type "JournalController" or select it from the
dropdown options. 

![link_table_view](/tuts_images/journal_custom_class.png)

Now we've associated our Journal Controller file with our UITableViewController 
in our storyboard. Our next step is to retrieve our entries from iCloud. 

{x: load_entries}
Add the following code to the JournalController class to retrieve entries from 
iCloud:

~~~language-swift
import UIKit

class JournalController: UITableViewController
{
    let entries = Entry()
    
    override func viewDidLoad()
    {
        entries.load()
    }

}
~~~

We first declare a constant and assign it to an instance of our Entry class. Then
when the viewDidLoad() function gets called, we use the load() method of our Entry
class to retrieve our saved entries. That's actually all we need to do to get our
saved entries. Let's add a bit more code so we can actually see some of the data 
we've gotten back.


{x: model_delegate}
Modify JournalController to conform to the ModelDelegate protocol: 

~~~language-swift
import UIKit

class JournalController: UITableViewController, ModelDelegate
{
    let entries = Entry()
    
    override func viewDidLoad()
    {
        entries.model.delegate = self
        entries.load()

    }
    
    func errorUpdating(error:NSError) {}
    
    func modelUpdated() {}
}

~~~

Our Entry class calls methods on the ModelDelegate in the completion blocks of
its asynchronous methods. We'll use those delegate methods to set our local 
variables. In order to conform to the ModelDelegate protocol, we need to implement 
two functions: errorUpdating() and modelUpdated(). 

Here we've also added a line in viewDidLoad() to set the model's delegate property
to self. 

{x: logging_entries}
Add one line to the modelUpdated() function to log the records property of our
entries object. 

~~~language-swift
func modelUpdated() {
    println("our saved entries are \(entries.records)")
}
~~~

If you build and run at this point, you'll see a log message of CKRecord objects
we retrieve in our Entry class' load() method when you tab over to the Journal view. 

{x: save_and_retrieve}
Save some entries and make sure you get some entries logged. You should see a log
message that looks like the the following: 

![logged_entries](/tuts_images/logged_entries.png)

## Getting our entries 

Now that we have gotten some data, let's populate the table cells in our UITableView
with that data. In order to populate our cells with our data, we need to implement 
two delegate methods. One tells the app how many rows we need in our table and 
the other tells the app what data to put in the table cell. It is important to 
note now that anytime we use the UITableViewController class, we automatically 
conform to the UITableViewDelegate and UITableVIewDataSource protocols. 

{x: row_and_data_methods}
Set the number of table rows to the number of entries we have by adding the following
code to our JournalController class:

~~~language-swift
override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int
{
    return entries.records.count
}
~~~


