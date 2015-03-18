# Entry Form Controller

## Introduction

Entering a new mood log is the main user action in Flourish. Therefore, it is no
surprise this view is the most interactive. In this section we will take our static 
entry form layout and add interactions to allow users to log their mood. We will 
also introduce IBAction methods and IBOutlet objects that we will use to link 
elements of our Storyboard to the code in our view controller. 

New Concepts This Chapter 
* Ternary Operator
* IBAction
* IBOutlet 
* Downcasting

## Creating EntryFormController.swift

{video: create_link_entry_form}

We've already created a view controller using storyboards in our interface 
chapter. What we want to do now is continue to work on that view controller
in code. To do that we need to create a .swift file that we then link to our
existing view controller in our storyboard. Here's how:

{x: new_swift_file}
Add a new swift file to your project. Go to file > new > file and select 
"swift file." Name the file EntryFormController.swift

{x: new_class_declaration}
Erase any code in the EntryFormController.swift file and replace it with the
following:

~~~language-swift
import UIKit
import CoreLocation

class EntryFormController: UIViewController, UITextFieldDelegate, UITextViewDelegate
{
    
}
~~~

The first line imports the UIKit library. The next line
is where declare a new class called EntryFormController that is an instance of
the UIViewController class. Next we have our class conform to the UITextFieldDelegate
and UITextFeldDelegate protocols. The protocols are going to allow us to access
the values typed into the our text field and text view objects. 

{x: link_to_interface}
Now that we've declared our class we want to link our new class to the our 
entry view controller in our storyboard file. Go to main.storyboard and click
on the entry form controller object in your entry view controller scene. Now go 
to that view controller's identity inspector and in the Custom Class section's class field,
type EntryFormController.

![link_custom_entry_class](/tuts_images/link_entry_controller.png)

That's it! We've now got our EntryFormController.swift file and our view controller
in interface builder linked up. Now we need to declare some basic methods in our
class. 


{x: controller_setup_methods}
Add the following to your EntryFormController.swift file.

~~~language-swift
 override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
~~~


## Mood Dropdown 

{video: flourish_picker_feelings_constants}

Let's start with what we need for our mood picker. Our end goal is to get our 
picker to look like this

![picker_image](/tuts_images/picker_example.png) 

Our dropdown will be built by 
placing the six mood option buttons in front of an image background. As always 
let's begin by defining our constants and variables. The first one we will define 
is a constant for our picker dropdown background. 

{x: create_picker_constant}
Add the following code to the top of your EntryFormController.swift file at the
top right after the class is declared. 

~~~language-swift
let picker = UIImageView(image: UIImage(named: "picker"))
~~~

UIImageView class is a container that allows us to display a single image or 
animate a series of images. In this case we want to display an image of type 
UIImage named "picker". The named:Name method of UIImage looks for an image file
of the same name in the assets folder of the application. If you spell the name
wrong, your app will still build and run, but you'll quickly find out your image
is missing! Watch for those spelling mistakes when referencing image names. 

Now that we have a constant for our background, we need the mood option buttons. 

{x: defining_mood_collection}
Begin by defining a collection that contains the all of our mood options. 

~~~language-swift
let feelings = [
  ["title" : "the best", "color" : "#8647b7"],
  ["title" : "really good", "color": "#4870b7"],
  ["title" : "okay", "color" : "#45a85a"],
  ["title" : "meh", "color" : "#a8a23f"],
  ["title" : "not so great", "color" : "#c6802e"],
  ["title" : "the worst", "color" : "#b05050"]
]
~~~

Our collection is an array type. It is really an array of dictionaries because each member of the 
array has a key-value pair. As with any array each member has an index which we will use for
our button tag, which will represent the value of the option. Low numbers will
represent a better mood and high numbers will represent worse moods. 
We will then use the array member at index to retrieve the button title and 
color, which we will use to create the button label.Once we have our constants, 
we will position and style our picker. 

{video: flourish_picker_hidden_alpha}

{x: picker_properties}
Start setting up the picker by typing following into the viewDidLoad() method: 

~~~language-swift
picker.frame = CGRect(x: 45, y: 160, width: 286, height: 291)
picker.alpha = 0
picker.hidden = true
picker.userInteractionEnabled = true
~~~

The first line sets the frame of property of our UIImageView. As we previously 
learned, UIImageView is a container for our UIImage, the frame property has the
position of the frame and the size. Our frame is going to be a rectangle, so we 
can use the CGRect data structure to represent the dimension and location of our 
frame rectangle. The x and y properties denote the rectangle's origin, while 
width and height denote size. It's worth mentioning now that iOS apps have a 
coordinate system that originates (point 0,0) from the upper left corner of the 
screen. 

Next we set hide our image view by giving the `alpha` property a value of 0 and 
setting the `hidden` property to true. With alpha, 1 is synonymous with 100% 
opaque, while anything between 1 and 0 will show our UIImageView in varying 
percentages of transparency. You might be wondering why we chose to reduce the 
alpha property to 0 <i>and</i> set the hidden property to true, seeing as how both 
of these properties make the dropdown invisible. The reason is, that an object 
with an alpha value of 0 is still drawn, and still takes up space in our view. 
It also means that, despite not being visible, it can still inhibit user 
interaction with any objects that are stacked behind it. In our case, simply 
setting the dropdown's alpha to 0 would mean it would block users from interacting 
with the textfield behind it. This would likely cause your users to get angry that 
the textfield is "broken." On the other hand, we cannot simply rely on the hidden 
property because it only has a boolean value to toggle appearance, and we would 
like to animate and fade in our dropdown. Thus, we will leverage alpha for the 
fade in affect and animate its value from completely transparent (0) to fully 
opaque (1).

Finally our last line sets the `userInteractionEnabled` property to true, which
means any touch events on our UIImageView will be registered, rather than 
ignored. Most of the elements in our storyboard have the userInteractionEnabled
property set to true by default. However, when we initialize a new UIImageView
object in code, the userInteractionEnabled property is set to false by default. We actually
could have drawn the picker in our Main.storyboard file using interface builder and 
set our properties attributes inspector. The reason we did not draw our dropdown
in the interface builder is because we are going to be problematically creating 
what's called <i>subviews</i> of the button.

{video: flourish_subview_superview}

Subviews and superviews refer to a parent-child relationship between views. The
superview is the container for the subview. This means that any coordinates a 
subview has, start from the upper left corner of the superview. It also means 
that any transparency effects on the superview affect the subview. Furthermore, 
it can mean the superview can clip subview contents that would otherwise overflow
the suprview's frame. In this view controller, our picker is going to be 
a subview of the view controller's view property and the mood buttons are going 
to be subviews of the picker. To see how this plays out, we first need to 
actually add the picker subview to the view controller's view property. 

{x: add_picker_subview}
Right below our previous code, add the picker subview to the view controller's 
view property by adding: 

~~~language-swift
view.addSubview(picker)
~~~

This adds a subview, picker, to our view controller's view property. All view
controllers have a view property, of type UIView, that can be referenced simply
as view. 

Now that we have created a new subview, let's make sure nothing has gone awry. 
Go back and change the picker's hidden property to true and alpha to 1 and build
and run to make sure our subview is showing up. You should see this:

![dropdown_no_button_example](/tuts_images/dropdown_no_button.png)

You might need to reposition some of your labels to get the tail of the picker
image to line up with your select button. 

{x: reset_alpha}
Ok we have our picker. Reset alpha to 0 and the hidden to true and let's move on. 

Now that we have the picker, we need to mood buttons that go inside the picker. 
For that we are going to add a subview to picker for each mood option. 

{video: flourish_ui_color_helper}

Before we can do that we need to add our first helper file. What this is going
to do is extend the UIColor class to accept hex strings and interpret them as 
rgba values. We could express our colors in rgba values throughout our app and 
forgo this step altogether, but hex strings are so common we'd rather make our
lives easier at this point. 

{x: helper_folder}
Create a new folder in your project navigator by going to file > new > group and 
calling this folder "Helpers".

{x: models_class}
Create a new file in your project navigator by going to file > new > file > 
swift file and name it "UIColorHelper.swift". Place this file in your Helpers folder.

{x: ui_color_code}
In UIColorHelper.swift, add the following code: 

~~~language-swift
import UIKit

extension UIColor
{
  convenience init(rgba: String)
  {
    var red:   CGFloat = 0.0
    var green: CGFloat = 0.0
    var blue:  CGFloat = 0.0
    var alpha: CGFloat = 1.0
    
    if rgba.hasPrefix("#")
    {
      let index   = advance(rgba.startIndex, 1)
      let hex     = rgba.substringFromIndex(index)
      let scanner = NSScanner(string: hex)
      var hexValue: CUnsignedLongLong = 0
    
      if scanner.scanHexLongLong(&hexValue)
      {
        if count(hex) == 6
        {
          red   = CGFloat((hexValue & 0xFF0000) >> 16) / 255.0
          green = CGFloat((hexValue & 0x00FF00) >> 8)  / 255.0
          blue  = CGFloat(hexValue & 0x0000FF) / 255.0
        }
        else if count(hex) == 8
        {
          red   = CGFloat((hexValue & 0xFF000000) >> 24) / 255.0
          green = CGFloat((hexValue & 0x00FF0000) >> 16) / 255.0
          blue  = CGFloat((hexValue & 0x0000FF00) >> 8)  / 255.0
          alpha = CGFloat(hexValue & 0x000000FF)         / 255.0
        }
        else
        {
          print("invalid rgb string, length should be 7 or 9")
        }
      }
      else
      {
        println("scan hex error")
      }
    }
    else
    {
      print("invalid rgb string, missing '#' as prefix")
    }
    
    self.init(red:red, green:green, blue:blue, alpha:alpha)
  }
  
  class func lighten(color: UIColor, percentage: CGFloat = 1.5) -> UIColor
  {
    var h: CGFloat = 0
    var s: CGFloat = 0
    var b: CGFloat = 0
    var a: CGFloat = 0
    
    color.getHue(&h, saturation: &s, brightness: &b, alpha: &a)
    
    return UIColor(hue: h, saturation: s, brightness: (b * percentage), alpha: a)
  }
}
~~~

Now anything we want to pass a string represented a color's hex code, we simply
call UIColor(rgba: hex_string). We'll see a concrete example shortly, when we 
write a method to create our loop. 

{video: flourish_picker_loop}

{x: add_button_loop}
In the viewDidLoad method, after your picker.userInteractionEnabled = true 
statement but before view.addSubview(picker) add the following: 

~~~language-swift
var offset = 21

for (index, feeling) in enumerate(AppHelper.properties.moods)
  {
    let button = UIButton()
    button.frame = CGRect(x: 13, y: offset, width: 260, height: 43)
    button.setTitleColor(UIColor(rgba: feeling["color"]!), forState: .Normal)
    button.setTitle(feeling["title"]!, forState: .Normal)
    button.tag = index
    button.addTarget(self, action: "setMood:", forControlEvents: .TouchUpInside)
    
    picker.addSubview(button)
    
    offset += 44
  }

~~~

In order to understand what is going on here, we need to revisit our goals. We 
want to stack mood buttons inside our picker image background so it looks like a 
dropdown menu. We can achieve this by spacing the buttons an equal vertical 
distance from the previous one using an offset variable. The offset variable
will represent the y coordinate of the button's frame property. Our initial offset
variable will start at 21, which is allows the first button we are going to create
to clear the dropdown image's tail. 

Now we are going to look at our for-in loop. Using Swift's native enumerate 
function returns a Tuple for each value in the array, giving us an index
variable we can use in our loop. Index refers to the iteration of the loop. Swift's
enumerate function index starts at 0 and increases by 1 each time. We use the 
index variable to set the tag property for each mood button we create. A button's 
tag is simply and identifier we can use later on to answer the question: "Which 
one of these buttons was pressed?" 

Now our for-in loop should be making sense. The first thing we do is declare a 
constant equal to a new instance of UIButton. Then we set the frame values for 
the position and the height and width of our new button. Next we set the font
color for our button's label as equal to the color property of the moods array
member we are currently iterating (note: we only care about normal state because
the picker will hide once the button is selected). After that we take the title 
property of that array member and set it as our UIButton's title text. 
Our next line gives the button a unique identifier via the tag property, while
the next line assigns an action "setMood" for the TouchUpInside event (we will
write our setMood() method later in this tutorial). A TouchUpInside event is fired 
whenever a user taps a button. Our penultimate line in our loop adds our button 
as a subview on the picker view. Finally, we increment our offset variable before 
our next iteration of the loop. 

We've actually just created a lovely picker, but before we can build and run to 
see our beautiful dropdown menu, we need methods that will show and hide our dropdown. 

{video: flourish_open_picker}

{x: open_picker_method}
Create an openPicker() method with the following code:  

~~~language-swift
func openPicker()
  {
    self.picker.hidden = false
    
    UIView.animateWithDuration(
      0.3,
      animations: {
        self.picker.frame = CGRect(x: 45, y: 180, width: 286, height: 291)
        self.picker.alpha = 1
      },
      completion: { finished in
      }
    )
  }

~~~

If you recall way back when we wrote the code for our picker in viewDidLoad(), 
we set it to hidden with an alpha value of 0. Naturally if we want to show it, we 
need to set the hidden property to false and give the picker's alpha property 
a value of 1. The first line of our openPicker method simply sets our picker's 
hidden property to false. Naturally, the next line could simply be:

~~~language-swift
  self.picker.alpha = 1;
~~~

But that's lame! Sure that will make the picker visible, but why do that when we 
can easily create a cool fade-in effect? 
Thought so. UIView has a great method called animateWithDuration() that takes as 
parameters a duration, an object of properties you want to animate, and a 
completion closure. In openPicker(), we are using 0.3 seconds as the duration of our 
animation and we'll be animating the alpha property as well as the frame property
of our picker. When we instantiated our picker, we set the frame rectangle's y
coordinate equal to 160. We can animate the picker's vertical position simply
by setting a different y coordinate for the picker in the animations object. 
Instantiating at y: 160 and ending at y: 180 is going to make the picker slide down
over the course of the animation. That, combined with our fade-in effect achieved
by going from alpha = 0 to alpha = 1 will make your users smile. 

You now might be wondering about completion closure. The completion argument 
of animateWithDuration simply denotes a block of code to run after the animation
has completed. In this case we aren't doing anything after the animation so we 
leave the closure body empty.

{video: flourish_close_picker_method}

{x: close_picker_method}
Now that we have an open picker method, we need a method with the converse of all
of our openPicker commands to have the picker fade out and slide up. Here's the
closePicker() method: 

~~~language-swift
  func closePicker()
    {
      UIView.animateWithDuration(
        0.3,
        animations: {
          self.picker.frame = CGRect(x: 45, y: 160, width: 286, height: 291)
          self.picker.alpha = 0
        },
        completion: { finished in
          if (finished) {
            self.picker.hidden = true
          }
        }
      )
    }
~~~

Ok let's take stock: we have created a picker object and given it buttons as 
subviews. We've added the picker (and by extension it's subviews) to our view 
and we wrote methods for opening and closing our picker. Now we need to call
the openPicker() and closePicker() methods to show and hide our picker when our 
app is running. 

{video: flourish_toggle_picker_method}

{x: toggle_picker}
To toggle our picker, write a method that calls closePicker() if the picker is open
and openPicker() if the picker is closed. Then we'll have a button in our interface
call that method when a user taps the button. Meet our togglePicker method: 

~~~language-swift
  @IBAction func togglePicker(sender: AnyObject)
    {
      picker.hidden ? openPicker() : closePicker()
    }
~~~ 

Our togglePicker() method uses a handy operator called the ternary operator (?). 
The ternary operator defines a conditional expression. Think of it as short hand
for and if-else statement.  The condition is on the left of the ternary operator, 
a true condition and false condition to the right of the operator separated by a
colon. Using our togglePicker function as an example, we read the ternary statement
as "if picker.hidden is true, call the openPicker method. If false, call the 
closePicker method." 

## IBActions 

You may have noticed that our togglePicker() method has an @IBAction deceleration
before our familiar func declaration. The @IBAction declaration is known as a 
"type qualifier" and it denotes our method is a special type: an IBAction. 
IBAction stands for Interface Builder Action and, as the name might suggest, is 
a method that gets triggered by an object in our interface builder. The object 
that calls the IBAction is known as the sender, which means we can query the 
sender object in our method. "AnyObject" is a protocol that can represent an 
instance of any class type.  
Read our function declaration line as "Our function is an IBAction and it is 
called togglePicker and it returns the sender object that called the function, 
which can be of any type."

Now we just have one more step before we take a break to build our project. We
need to connect our select button in our storyboard to the togglePicker method.

{x: connect_toggle_to_storyboard} 
To connect an object to an IBAction, the easiest way is to open up our main.storyboard
file next to our EntryViewController.swift file and dragging an outlet from 
the object to the IBAction. To get a side-by-side of two files, simply click on
the Assistant Editor and open the two files we know. Now in our main.storyboard
file, right click on the select button to reveal a menu of sent events. Right 
click on the touchUpInside event and drag an outlet to the togglePicker method. 

We've now set the togglePicker method to be called whenever our select button's
touchUpInside event fires, i.e. when a user taps the button. 

{x: build_check_toggle} 
Now finally, build and run. Tapping on the select button will reveal toggle our
picker with the mood buttons we created! 

Now we need to write some more code to  allow us to set a mood when we tap a 
mood button. 

###IBOutlet 

{video: flourish_feeling_button}

{x: declare_feelingButton_IBoulet}
To reflect a set mood, we are going to change the label of our select button in 
our interface builder to show the mood we've chosen in our picker. Write the 
following below your other variable and constant declarations: 

~~~language-swift
  @IBOutlet weak var feelingButton: UIButton!
~~~ 

A few things to note with IBOutlet declaration, starting with the weak designation. 
IBOutlets can be strongly or weakly referenced. For this tutorial, all you need
to know is that subviews should be weakly referenced and top level views should
be strongly referenced. Since our button is a subview of the top level view, we
use weak for our deceleration. After that we simply declare the variable as normal,
but you need to be aware that IBOutlet variables are all optionals. That means 
when we declare them, we should implicitly unwrap the variable's value by placing
the exclamation mark after the variable type. Implicit unwrapping, in case you 
forgot, tells xcode "assume this always has a value, even though it is an optional."
This means from now on we can simply reference the variable name feelingButton 
and its value will be unwrapped each time we use it. Otherwise we'd have to 
explicitly unwrap the optional's value each time we use it in our code. 

Now that we've declared our IBOutlet variable, we need to link it to our interface
builder object to our IBOulet variable. This works very similar to linking IBActions. 

{x: connect_feeling_to_storyboard} 
With both our main.storyboard file and our EntryFormController.swift files open, 
Ctrl-click on the select button in the storyboard and drag it onto the feelButton
variable declaration. 

{video: flourish_set_mood_method}

Now we have access to our interface builder button in our controller, which means we can change
its label property to our selected mood. Back in our EntryFormController,lets 
write our setMood() method to accomplish this: 

~~~language-swift
  @IBAction func setMood(sender: AnyObject)
  {
    feelingButton.tag = sender.tag
    feelingButton.setTitle(sender.currentTitle, forState: .Normal)
    feelingButton.setTitleColor(sender.titleColorForState(.Normal), forState: .Normal)
    
    closePicker()
  }
~~~ 

In setMood() we are updating the tag, currentTitle, and titleColorForState 
properties of our feeling button to the tag, currentTitle, and 
titleColorForState properties of the sender. The last line in the method calls 
our closePicker() method, which hides our picker. Recall that we have already
linked this IBAction to the touchUpInside event of our mood buttons in the for-in
loop where we created the mood buttons in the first place. 

{x: build_run_setting_mood} 
Ok now build and run, you'll notice that when you pick a mood from the picker 
dropdown, our feelingButton's label changes from "select" to the title of the 
mood you've selected!

## Getting location data 

We want to be able to allow the user to enter location data for their entry. This
can help our user determine if certain places are more helpful or hurtful to mental
health. In this section, we are going to get permission from the user to use
location services so we can store the user's location along with the other 
journal entry data. 

{video: flourish_cllocationmanager_plist}

{x: adding_core_location_libary}
First thing we need to do is import the Core Location library. You'd now have
the following two import statements in EntryViewController.swift

~~~language-swift
import UIKit
import CoreLocation
~~~

In iOS, the CLLocationManager class oversees every aspect of location services. 
This is going to be the primary class we use to get our user's location to add
to our journal entry. Let's put this class to work:

{x: instantiate_locationManager}
In EntryViewController.swift, declare variables for location manager and current 
location. We aren't going to assign our variables to anything yet, but we are
going to specify that locationManager be of type CLLocationManager and 
currentLocation is of type CLLocation. 

~~~language-swift
  var locationManager: CLLocationManager!
  var currentLocation: CLLocation?
~~~ 

Notice the CLLocationManager type is an implicitly unwrapped optional because
we are going to assign it to an instance of the CLLocationManager class, therefore
it will always have a value.  Our currentLocation variable is an optional because
we are only going to have a value if the user grants our app permission to 
record location data. 

{x: setuplocation_method}
Now create a new method called setupLocationManager() to initialize our location
services with the following code:

~~~language-swift
func setupLocationManager()
  {
    locationManager = CLLocationManager()
    locationManager.delegate = self
    locationManager.requestWhenInUseAuthorization()
    locationManager.desiredAccuracy = kCLLocationAccuracyBest
    locationManager.distanceFilter = kCLLocationAccuracyNearestTenMeters
    locationManager.startUpdatingLocation()
  }
~~~

The first line of setupLocationManager() assigns our locationManger variable to
an instance of the CLLocationManger class. The second line assigns the 
locationManager delegate to our view controller. The delegate is going to receive
all of the location-related information. The third line is the requestWhenInUseAuthorization()
method, which will prompt the user to allow our app to use location services. Next
we set some preferences for our location data. 

The desiredAccuracy property allows
us to tell our receiver when to give us updated location data. For example, if we
needed to get kilometer-level data, we could set the desiredAccuracy property to 
kCLLocationAccuracyKilometer. This would say "don't talk to me until you have
kilometer-level location data because that's the level I need." It's great to 
be able to set a standard of accuracy, but we aren't relying on location data for 
anything fancy like walking directions. We want to get our location data as 
quickly as possible so we're simply going to set our desiredAccuracy to the
default value: kCLLocationAccuracyBest. 

Next we set the distanceFilter property 
of the CLLocationManager class, which sets the minimum distance  a device must
move horizontally before an update event is generated. We're going to request an 
update at every 10 meters of horizontal distance, so we set our property to 
kCLLocationAccuracyNearestTenMeters. It's important to note there isn't an exact
science for choosing your desiredAccuracy and distanceFilter properties. Just
know that the more accurate you want your location data, the more hardware 
resources you're consuming. Finally we call the startUpdatingLocation method 
which returns immediately an initial location to our locationManager delegate. 


Speaking of the delegate we need to make our view controller conform to the
CLLocationManagerDelegate protocol to clear the error xcode should now be throwing. 

{x: CL_delegate_declaration} 
Add the CLLocationManagerDelegate protocol to our list of protocols in our class
declaration. 

~~~language-swift
class EntryFormController: UIViewController, UITextFieldDelegate, UITextViewDelegate, CLLocationManagerDelegate {
~~~

{x: location_in_viewdidload} 
We want to get our location data as soon as a user begins entering a new mood so
add the call to the setupLocationManager() method in the viewDidLoad() method.

{x: location_in_plist} 
Add a NSLocationWhenInUseUsageDescription key in 
Info.plist with a our message as a string. We need to do this to set a message to get 
displayed to the user when we ask for permission to use location services.

![adding_plist_location_permission](/tuts_images/location_permission_plist.png)

{video: flourish_location_delegate}

Now we've got our location services set up. The user will be prompted to allow
our app to use location services. Now we need to handle location events and data. 
The first thing we need to handle is a change in the authorization status of our
location services. The user can authorize our app and change his/her mind later
and deauthorize it. We want to be made of aware of any change in our authorization
status.

{x: didChangeAuthorizationStatus_function}
Create a method for responding to a change in location authorization by using the
didChangeAuthorizationStatus() method of the CLLocationManagerDelegate class:

~~~language-swift
 func locationManager(manager: CLLocationManager!, didChangeAuthorizationStatus status: CLAuthorizationStatus)
    {
      switch status
      {
          // User has not yet made a choice with regards to this application
      case .NotDetermined :
          manager.requestWhenInUseAuthorization()
          println("prompt the user to enable location services")
          
          // User has explicitly denied authorization for this application, or
          // location services are disabled in Settings.
      case .Denied :
          println("prompt the user to re-enable location services in settings")
          
          // User has granted authorization to use their location only when your app
          // is visible to them (it will be made visible to them if you continue to
          // receive location updates while in the background).  Authorization to use
          // launch APIs has not been granted.
      case .AuthorizedWhenInUse :
          manager.startUpdatingLocation()
          println("authorized when in use")
          
          // Currently only .Restricted falls here and there's not much we can do about it
          // so we'll simply move on with our lives
      default :
          //do nothing
          println("Other status")
      }
    }
~~~

We've added some comments to explain this switch statement. The general idea is to
respond to a change in authorization status. What is important to know is that
we our running our switch on the status parameter, which can take a few forms:
.NotDetermined, .Denied, and .AuthorizedWhenInUse. In cases where we don't have
authorization, we want to ask for it using the requestWhenInUseAuthorization() 
method of our CLLocationManager. When we get permission, we want to call the 
startUpdatingLocation() method to receive location data. 

Now we can finally get our payoff! Here's how to capture the location data we 
get:


{x: didUpdateLocations_function}
Create a method for responding to a location update event by using the 
didUpdateLocations() method of the CLLocationManagerDelegate class.


~~~language-swift
func locationManager(manager: CLLocationManager!, didUpdateLocations locations: [AnyObject]!)
  {
  currentLocation = (locations.last as! CLLocation)
  }
~~~

The didUpdateLocations() will give us access to the locations array that contains
all of the objects containing location data. Each time we get new location data, 
a new location of type CLLocation gets pushed onto the locations array. There
is always at least one object in the locations array, but it can have many if updates were received
but not delivered or if the updates were deferred. Regardless, we only care about the 
most recent current location so we want to access the last member of the 
locations array. Swift has a built in shortcut for the last member of the array, 
.last. 

The  "locations.last as! CLLocation" part of our function might be confusing so 
let's go over it.  That is an example of forced downcasting. Downcasting is used when we are unsure 
if a constant or variable refers to a certain type. When we are unsure we can try
to treat it as an instance of a type using by using the as operator. Downcasting, however, 
is pretty risky because of the uncertainty inherent in not knowing the type of the
thing you are trying to downcast. If, for example, our variable is an array type 
and we try to cast it as an a integer, our cast will fail. When unsure, it is
best to use the optional form of the as operator (as?) which will cast to the 
type we specify or return nil if the cast fails. In this case, our locations parameter
is AnyObject, which doesn't tell us anything about its type, so you might be 
thinking we'd use the optional as operator to downcast to a CLLocation object. 
However, looking at the documentation for [the didUpdateLocations event](https://developer.apple.com/library/mac/documentation/CoreLocation/Reference/CLLocationManagerDelegate_Protocol/index.html#//apple_ref/occ/intfm/CLLocationManagerDelegate/locationManager:didUpdateLocations:)
we can see that the locations array only contains CLLocation objects. That means
we are certain that locations.last will always refer to a CLLocation object and 
we can avoid using the as? operator. Instead, we used forced downcasting by using
the forced form of the type cast operator (as!) and we wrap the downcast 
statement in parenthesis to tell xcode we really really mean to use as! as 
opposed to as?. 


