# Designing our Interface

## Introduction

In order to kick off our project, we need to design our user interface (UI). By the 
end of this section we'll have all of the views we need for Flourish set up and 
we can spend the rest of the tutorial adding interactivity and functionality. 
Thinking back to our MVC section, we are now creating views. This means we'll have
mostly static UI elements that will later become more interactive and dynamic
when we create controllers for each view. In Xcode, the easiest way to create 
our UI is using storyboards. Storyboards are a visual tool for laying out 
application elements using a drag and drop interface and a series of menus. 

New Concepts This Chapter 
* Scenes
* View Controllers
* View Stacks
* Controller Nesting
* Auto Layout

{video: flourish_scenes}

{x: explore main.storyboard}
Jump to your main.storyboard file and let's get acquainted. When we created our project, 
we said we were making a tabbed application, which means Xcode has already given 
us a few things in our storyboard. First, notice we have several labeled boxes
with connectors between them. Each box represents a scene.  

![scene_img](/tuts_images/scene_img.png)

A scene consists of a view and a view controller. The view is represented visually
and is what we're going to drag elements onto. The view controller will be a
.swift file where we'll later add things programmatically. Before anyone gets 
confused let's clear something up: everything in a storyboard can be done
in code. Our storyboards are just a visual way for us to create swift code.

Now looking at our storyboard sidebar, we have a list of three scenes: First Scene, 
Second Scene, and Tab Bar Controller scene. Generally, each view controller in
our storyboard is going to have a .swift file associated with it for us to add
code pertaining to that storyboard. However, looking at our project navigator, 
you see we have FirstViewController.swift and a SecondViewController.swift file 
and no file to correspond to the Tab Bar controller in our storyboard. We will
eventually add a file for this controller, but there is still a controller in our
project. Elements that are dragged onto a storyboard have default functionality
and if we are not extending that functionality, we often don't need to create a
.swift file for that controller. 

{video: flourish_build_and_run}

{x: first_build}
Hit the play button in the upper left hand corner your Xcode window to build and 
run the project. 

This will compile our code into an app and install that app onto
the device specified in the build scheme. You can edit the target device by 
selecting the dropdown menu in the active scheme editor. 

![play_button](/tuts_images/play_button.png)

{x: set_target}
Let's select iPhone 6 as our target device and run our app. This will launch our 
iOS simulator in a separate window and run our app. You'll see  our scenes and 
the tab bar that we can use navigate between them. 

Now that you're comfortable 
with what xcode gives us as a default, let's go back to xcode and start adding 
views. 

## Nesting View Controllers

{video: nesting}

It is good practice in iOS to encapsulate functionality in its own view controller 
to keep our controller code short and focused. With that in mind, we want to 
introduce the concept of a "container" view controller. A container view controller
is no different from any other view controller (called a "content" view controller)
except that the container view controller's sole purpose is to present other view
controllers. 

{x: container_view_controller} 
Read the ["Designing Your Container View Controller"](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/CreatingCustomContainerViewControllers/CreatingCustomContainerViewControllers.html) section in the developer docs.

Placing a view controller inside another is called nesting and our current app
design could benefit from it. The way we are going to approach this is to
create container controllers for each of our tab views and we are then
going to create content controllers within each. It is important to note at this
point that you can always nest a controller within a controller and there are few
hard and fast rules about when to nest. 

However, there are some controllers that 
are almost always container controllers and should <strong> always </strong> 
have a nested content controller if you'd like to display content within that 
controller. One such controller is a navigation controller, whose purpose is to 
manage navigation of hierarchical content. We are actually going to use a 
navigation controller as our container view because some of our tabs are going 
to have views that we navigate to from the initial view. Navigating "down" from an 
initial view to other views and hitting a back button to get to the previous view
is called view stacking and we can visualize a physical stack of views. 

![view_controller_stack](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Art/navigation_interface_2x.png)

<i> this image shows a stack of views. Notice the back buttons to return
to the previous view in the stack</i>

Our navigation controller is going to manage our view stack. 

Let's do an example: 

{x: journal_navigation_controller} 
In our main.storyboard file, you'll want to select the object library in the
lower right corner of your screen. Drag a navigation controller from the object 
library and drop it anywhere in your storyboard. This creates a new scene in your 
storyboard. Like all scenes, our new scene has a view controller and a view. 

The navigation controller object you've just added to your storyboard actually
contains two controllers: the container controller and the first content controller
in the stack. The first content controller in a stack is called the "root" view
controller. 

{video: flourish_segue_tab}

{x: tab_controller_relationship} 
Ctrl-click on the Tab Controller Scene and drag an outlet onto your new navigation
controller scene. When you drop the outlet on your new controller, a menu will appear
asking you to establish a relationship between your Tab Bar Scene and your new
view controller scene. Under "relationship segue", select the "view controller"
option. Once you do that you'll notice your tab bar controller now has a third tab 
option.

![adding_segue_navigation](/tuts_images/adding_segue_navigation.png) 

![adding_tab_item](/tuts_images/adding_tab_item.png)

{x: removing_default_table_view} 
Sadly our default root view controller in storyboard is a TableViewController, 
which isn't what we need. Click on the root view controller scene and delete it 
entirely. 

{x: add_view_to_nav} 
Drag a plain ol' view controller object from the object library and drop it onto the 
storyboard and ctrl-drag an outlet from the navigation controller scene to the 
new view scene. When you drop the outlet, select "root view controller" from the 
Relationship Segue menu. 

![view_nav_outlet](/tuts_images/view_nav_outlet.png)

![view_nav_connection](/tuts_images/view_to_nav.png)

{x: adding_label} 
Let's add a label to this new view by dragging a label object from the
object library and drop it onto our new navigation controller scene's root view.
Next, double click on the label to change the label text to "Entry view." 
After that, double click on the tab bar item's name and change the name from item 
to Entry. 

{x: check_third_view}
Build and run your project to confirm we've added a third view. 

{x: delete_default_controllers} 
The default scenes in our storyboard aren't set up the way we want so go ahead
and delete the First Scene and Second Scene scenes from the storyboard. You
should now only have three total scenes: a tab bar controller, a navigation controller,
and a view controller. 

![one_view_left](/tuts_images/first_view_screenshot.png)

## Adding the other views to our default project 

{video: adding_tabs}

Our app is going to consist of 6 total views: 
* Authentication
* Entry
* Journal
* Calendar
* Trends
* Settings

All of these views, with the exception of the authentication view, are going to 
be tabs in our tab bar. Currently our tab bar only has one view, so let's add 
four more.  

{x: adding_calendar_trend} 
Follow the same procedure as we did for our Entry scene to create the remaining
views: Journal, Calendar, Trends, and Settings. 

Your tab bar now should have five items and you should have a total of five 
navigation controller/view controller pairs. 

![all_tabs](/tuts_images/all_tabs.png)

## Adding images to tab bar items 

{video: adding_tab_icon_names}

We are now going to add our custom tab icons to our tab bar. Xcode image assets
are all stored in a images.xcassets folder, which you can find in the project 
navigator. 

{x: download_icons}
Open up images.xcassets and you'll see the square and circle images
that are used in the default tab bar icons. This isn't an illustration tutorial, 
so you can go ahead and download our 
[icon pack](https://dl.dropboxusercontent.com/u/80807880/Icons.zip). Unzip the
file and you'll have an icons folder. 

{x: add_icons}
Drag the icons folder into your xcode 
project's images.xcassets. You'll now see your icons in the images.xcassets folder. 

![tab_icons_added](/tuts_images/adding_tab_assets.png)

If you are adding your own icons, please follow the [Icon and Image Sizes 
guidelines](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/IconMatrix.html) 
from Apple. 

Now that we have our image assets in our project, we can go back to main.storyboard
and assign some of those images to our tab bar items. 

{x: add_tab_image}
In your entry scene, click
on the tab bar item. Once selected, turn your attention to the attributes 
inspector's "Bar Item" section. There, you'll see an image field with a dropdown
menu. The dropdown options correspond to the names of your image assets in your
image.xcassets folder. If you're using our icons, select the "edit" icon for this
tab bar item. 

{x: repeat_tab_image}
Now repeat this process to select the appropriate image for the
rest of the tab bar items. 

The images for each tab bar item are as follows:
* Entry Scene - edit icon 
* Journal Scene - contacts icon
* Calendar Scene - calendar icon
* Trends Scene - pulse icon 
* Settings Scene - settings icon 


## Adding UI Elements with Auto Layout 

If you build and run our project so far, you'll notice label we added
to our entry view tabs appear in a different place than where we dropped it into the
scene. This is because we haven't really positioned our label yet. In order to 
make it easier to design interfaces for multiple screen sizes, the wiz kids at 
Apple gave us Auto Layout. As Apple describes "Auto Layout is a system that lets 
you lay out your app’s user interface by creating a mathematical description of 
the relationships between the elements. You define these relationships in terms 
of constraints either on individual elements, or between sets of elements." 

{video: flourish_auto_layout_text_field}

{x: auto_layout}
Before moving on, familiarize yourself with the ["Auto Layout menus in Xcode"](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/WorkingwithConstraints/WorkingwithConstraints.html#//apple_ref/doc/uid/TP40010853-CH8-SW3) 

Now let's go through a concrete example of auto layout by creating our new entry
form UI. Here's a wire frame for this view:

![x: entry_wireframe](/tuts_images/flourish_entry_wireframe.png)

As you can see we have a text field for entering a title for your entry, a
select button to select a mood from a set of options, a text field for the body
of your journal entry, and a save button to save your journal entry. Let's select
our new entry scene and begin building our interface. 

{x: remove_entry_label}
The "entry scene" label has served its purpose. Go ahead and delete it. 

{x: background_color}
Select your Entry view and change the background color to a dark blue in the attributes inspector.
If you're wondering, we used RGB(21, 69, 110) for the production app. 

{x: entry_height_constraint}
Drag a text field object from the object library into your New Entry view and 
add a 40 point height constraint in the Pin menu.

{x: set_date_text}
Change the text property of our text field to "Date" in the attributes inspector.

![date_height_constraint](/tuts_images/date_field_height_constraint.png)

Make sure to select "items of new constraints" from the update frames dropdown in
the Pin menu. This option sets these constraints only the selected objects in our view, 
i.e. just our text field. 

Now we want to set a width for our date field. We can set a width constraint, but
that isn't a good idea. The goal of Auto Layout is to make our interface flexible
for different screen sizes, so some of our elements need to resize on some devices.
As a rule of thumb for resizeable elements <strong> only set height constraints 
in points, and set relative width with leading and trailing margin.</strong> We 
want our interface to work on iPhone 5, 6, 6 plus, and even some iPads. Those
devices vary wildly in width. Also, we want to have a good landscape mode as well
for all of those devices. Setting a relative width helps us avoid issues on 
different screens. 

{x: text_field_container_constant}
Ctrl-drag from the date field to the containing view and select leading space to 
margin. In the date field's size inspector, edit the leading space to superview
constraint to equal 40. 

![Ctrl_drag](/tuts_images/text_field_ctrl_drag.png)


{x: text_field_container_constant}
Ctrl-drag from the divider to the containing view and this time select trailing 
space to margin. In the date field's size inspector, edit the trailing space to 
superview constraint to equal 0. 

The previous two steps have set our date field to have a 40 point left margin and
to fill the rest of the horizontal space in the container. 

{x: text_field_container_constant}
Ctrl-drag from the text field to the top layout guide view and vertical space option.
In the date field's size inspector, edit the top space to: top layout guide constraint
to equal 15 points. 

Here's what your date field's constraints should look like in the size inspector:

![date_constraints](/tuts_images/date_field_constraints.png)

Build and run at this point using various hardware emulators to verify our 
constraints are working properly. 

Now we want to add an icon for this text field. In order to that, we need to 
add an image view object to our view. 

{video: flourish_auto_layout_title_icon}

{x: add_icon}
Drag an image view from the object library and drop it onto your entry view. Set a
height constraint of 13 points and a width constraint of 18 points in the pin menu. 

{x: add_background_image}
Change image property of our new image view to the pencil icon, which can be 
downloaded and added to your project [here](https://dl.dropboxusercontent.com/u/80807880/UI.zip). Feel free to also use your own icons if you want. 

We now want to horizontally align the center of our image with the center of our
text field. This is going to require some math. We have a 40 points tall date 
field with a 15 point top margin. Our icon, however is only 18 points tall and 
needs a top margin that will ensure its center is always aligned with the divider. What 
we can do is simply take half of the difference in heights between the date field and the
icon and add that to the date field's top margin. Our date field is 22 pixels 
taller with a 15 point top margin. That means we need a 26 point top margin on 
our icon (0.5 * 22 + 15).

{x: add_icon_top}
Set a 26 point top margin to the icon by ctrl-dragging from the icon to the 
container view and selecting "top space to layout guide."

{x: add_icon_left}
Add a 10 point leading space to container using the same ctrl-drag technique we've been using. 

Here's what our view should look like now. 

![ui_field_icon](/tuts_images/ui_icon_field.png)

Now we need to add a divider that is going to separate our date field from the 
other input objects in that app. 

{x: add_divider}
Drag an view from the object library and drop it onto our view. Set a
height constraint of 1 point. 

{x: divider_left_right}
Set a leading and trailing margin constraint of 0 between our Divider and its
containing view. 

{x: rename_divider}
Label the UIView as "Divider" by either selecting the view in the Document outline and hitting enter or changing it's label under Identity Inspector > Document > Label.

So far we've established a relative width of our divider with leading and 
trailing constraints. Now we need to give our divider a vertical position. 
Heretofore we've set a top margin between our elements and the top layout guide. 
For the divider, however, we are going to set a margin relative to the date
field, not the top layout guide. 

{x: divider_top}
Set a top for the divider relative to the date field. In the document outline, 
Ctrl-drag from the divider to the date field and select "vertical spacing" option.
In the divider's size inspector, change the top space to: Date constant to 
1 point. 

Here's what the divider's size inspector should now look like:

![divider_constraints](/tuts_images/divider_constraints.png)

{x: date_field_background}
In the attributes inspector, we now want to change our date field's border style to 
transparent and our font color to white in the date field's attribute inspector.

Here's what our UI looks like at this point: 

![ui_text_divider_icon](/tuts_images/ui_icon_field_divider.png)

## Finishing the Entry Form UI 

{video: flourish_auto_label_select}

{video: flourish_finishing_entry_layout}

We've gone over setting Auto Layout constraints between an object and its container
as well as between an object and another object. At this point, we aren't going 
to rehash these techniques for each part of the UI. You are armed with enough
knowledge to build it out yourself. Take a look at the following image for all
of the objects in our view to know which objects to grab from the object library. 
From there you can position them in the view. 

{x: auto_layout_issues}
Since you'll probably make a few mistakes on your own, read about [resolving Auto Layout Issues](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/ResolvingIssues/ResolvingIssues.html#//apple_ref/doc/uid/TP40010853-CH17-SW1) 

{x: build_out_entry}
Build out the rest of the New Entry scene interface using our Auto Layout 
techniques. It's ok to not have the exact same values as found in the source 
code. Just watch out for constraint conflicts! 

![ui_guide](/tuts_images/ui_guide.png)

## Tackling other views

Now that you understand how to use Auto Layout to position elements in your view,
you can build out the rest of the static UI. Some of the remaining views will 
make heavy use of interface builder and you'll get to practice your auto layout
skills. Other views, however, will mostly be constructed programmatically in 
our view controller code. Each of the remaining UI construction will be detailed
in the chapters for the individual views. 


