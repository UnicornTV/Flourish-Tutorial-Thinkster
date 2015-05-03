# Putting it all together

## Introduction
In this chapter we are going to create controller methods that will save an entry
to our database using the custom class methods we created in the previous chapter.
There are no new concepts in this chapter, we are just tying things together in
familiar ways. 

{video: flourish_first_save}

## New IBOutlets

Let's go back to EntryFormController.swift. Now that we have set up our model
and created methods in our Entry class to do basic CRUD operations to our model, 
we can call those methods from the controller. Our goal is to take the user inputs
in our entry form and save those to iCloud. So first things first we need to 
capture user inputs as variables in our controller. 

{x: text_input_outlets}
In EntryFormController.swift, create user input variables with the following code:

~~~language-swift
@IBOutlet weak var titleInput: UITextField!
@IBOutlet weak var bodyInput: UITextView!
~~~

{x: text_input_connection}
Connect the UITextView in the entry form storyboard to the bodyInput IBOutlet 
variable and connect the UITextField in the entry form storyboard to the 
titleInput IBOutlet variable. 

![input_outlet_connections](/tuts_images/enable_query_sort.png)

Now we can use the create method of our Entry class to save our entry.

{x: save_form_method}
Create an IBAction method called saveForm with the following code:

~~~language-swift
@IBAction func saveForm(sender: AnyObject)
{
    let entry = Entry(title: titleInput.text, body: bodyInput.text, mood: feelingButton.tag, location: currentLocation)
    
    entry.create() { success, message, error in     
        // Reset and clear the form if it saved
        if success
        {
            println("success!", message)
            self.titleInput.text = nil
            self.feelingButton.setTitle("select", forState: .Normal)
            self.feelingButton.setTitleColor(UIColor.greenColor(), forState: .Normal)
            self.bodyInput.text = nil
        }
        else {
            println(error)
        }
    }    
}
~~~   

By now this method should be relatively easy to understand. We are declaring an 
IBAction function called saveForm and we are passing in the sender object that
can be of any type. Inside the function we declare a local constant that takes
the value of an instance of our Entry class, which we initilize using our 
convenience initializer. We set the params of convenience init to the IBOutlet
variables from our entry form. 

Next we call the create method of our Entry class and we create two scenarios in
our completion block. If we have an error, we log the error. If succeed in 
creating our entry, we log a success message and we reset the form by setting all
of our IBOutlet form variables to nil. We also change the feeling button text
to red just to give a little visual feedback to the user.

{x: text_input_connection}
Connect the touchUpInside event of the submit button in your storyboard interface
to the saveForm IBAction variable in EntryFormController.swift.

{x: log_in_to_icloud}
Build and run your app in the simulator. Once running, log in to iCloud in your
simulator by going to settings > iCloud.

{x: save_entry}
Open up your app in the simulator and fill out the form. Hit the save entry button. 


If you got the success message...YOU ARE AMAZING. If you got an error message, 
try to debug it until you get success. 