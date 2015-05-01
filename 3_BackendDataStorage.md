# Saving our Data using CloudKit 

New Concepts This Chapter 
* Cloud Kit
* Convenience Initializer
* Singleton Pattern
* Getters and Setters
* Data Encryption
* Grand Central Dispatch

## Introduction
We've heretofore focused on the view and controllers aspect of the MVC framework. 
We've shown how to layout an interface in storyboards and how to add functionality 
in a view controller file. You can currently create a journal entry, but you can't
save it. What good is that? This is where the M of the MVC framework comes in. We
are going to set up a data model that represents our journal entries and use the
Cloud Kit framework to store our data in a cloud database and retrieve it whenever
we need to. 

## What is Cloud Kit? 

Cloud Kit is a framework that makes it easy for us to move data between our app
and iCloud containers. iCloud, of course, is Apple's cloud storage solution that
is free for up to 5gb of storage. Cloud Kit allows us to set up a data model and
has a database component, but as you'll realize quickly, Cloud Kit has done a lot
of the hard work for you when it comes to data mapping and has even provided a 
simple GUI for creating our data models. 

## Why Cloud Kit?

## Enabling Cloud Kit in Our App

{video: flourish_cloudkit_capability}

{x: cloud_kit_build_setting}
Choose the xcode project from the project navigator and navigate to the capabilities
pane. Toggle iCloud on in the capabilities pane.

Toggling on the iCloud capabilities tells xcode to configure our app to use iCloud.
An example of what xcode does for us it creates code signing and provisioning 
assets for you as you need them. 

{x: choose_account}
When you toggle on iCloud capabilities, you're prompted to select a developer
account to associate with this app. 

![choose_icloud_account](/tuts_images/choose_account_icloud.png)

{x: services_checkboxes}
In the iCloud menu of the capabilities pane, check the CloudKit box in the services
menu and make sure the key-val storage box is also checked. 

{x: containers_checkboxes}
Right below services is the containers menu. Select the "specify custom containers"
radio button and select the checkbox that corresponds to your bundleID. You should
see you are all set up by looking at the steps below the containers section. 

{x: fixing_errors}
There a few different errors that can occur when enabling iCloud capabilities. 
Usually hitting the "fix error" button fixes it. Example below:

![icloud_error_example](/tuts_images/icloud_error.png)

## Defining our Schema

{x: log_in_cloud_kit}
To begin creating a data model, log into your [cloud kit developer dashboard](https://icloud.developer.apple.com/dashboard/)
or click the "cloudkit dashboard" button in the iCloud pane of the Capabilities 
menu. 

Once you log in you'll notice a three-pane layout. The name of the iCLoud container
you are currently viewing is in the upper left corner. 

{x: correct_container}
Click on the name to bring
up a list of all iCloud containers in your developer account. Find the name that
matches the name of the container you specified under "containers" in the 
capabilities menu of your xcode project. 


Once you've gone to the right container, you'll notice by default the "record types" 
option of the schema menu is selected and there is a default record type called users. 
Anyone that has an iCloud account gets stored as a user. We need to create an additional 
record type called entry. 

{x: create_model}
Click on the add button (+) in the top left corner of the detail pane. 

{x: new_record_name}
Enter "entry" as the name for your new record type. 

We now have a new data model. It's time to add attributes to our data model to 
ensure that we are saving all of the essential information for our entries.

{x: new_record_name}
In the detail pane, select the "add attribute" link and add an attribute with 
an attribute name of "Body." The attribute type is String and we can leave the 
index checkboxes all checked. 

{x: other_attributes}
Add Title, Location, and LocalStorageID attributes to your Entry record type 
and select the attribute type and index checkboxes for each to match the image below:

![entry_record_table](/tuts_images/entry_record_table.png)

## Model, User, and Entry models

Now that we've defined our schema, let's hop back into our xcode project and write
some code.

{x:models_folder}
Create a new folder in your project navigator by going to file > new > group and 
calling this folder "Models".

{x:models_class}
Create a new file in your project navigator by going to file > new > file > 
swift file and name it "Model.swift". Place this file in your Models folder. 

{x:model_import_cloudkit}
In Models.swift, import Cloud Kit right below the import Foundation statement in
the file. 

~~~language-swift
import Foundation
import CloudKit
~~~

Our models.swift file is going to contain a protocol and a new class declaration. 

{x: model_delegate_protocol}
Add the following code to create a ModelDelegate protocol.

~~~language-swift
protocol ModelDelegate
{
  func errorUpdating(error: NSError)
  func modelUpdated()
}
~~~

Our ModelDelegate protocol requires conforming classes to all implement a standard
set of methods for pulling data from our cloud database. This protocol contains two
methods: errorUpdatng(error:NSError) and modelUpated(). We'll use errorUpdating(error:NSError)
to handle errors and we'll call modelUpdated() when more than one entry is refreshed.

{x: define_model_class}
Add the following code to define a new class called Model.

~~~language-swift
@objc class Model
{
  class func singleton() -> Model
  {
    return globalModelSingleton
  }
  
  var delegate: ModelDelegate?
  let container: CKContainer
  let publicDB: CKDatabase
  let privateDB: CKDatabase
  
  init()
  {
    container = CKContainer.defaultContainer()
    publicDB = container.publicCloudDatabase
    privateDB = container.privateCloudDatabase
  }
}

let globalModelSingleton = Model()
~~~

In the first line, we've declared a class called Model. We use the @objc keyword
to expose our swift class to our objective-c classes, in case we need to import 
that class as an objective-c header file. Lines 8-12 create our properties 
for our model class. The delegate property will refer to the class that conforms
to our newly-defined ModelDelegate protocol.  Container, publicDB, and privateDB
are basically name spaces for CloudKit methods and properties. 

In our initializer method, init(), we assign values to our non-optional properties.
Our container constant is assigned to CKContainer.defaultContainer() which returns
the container object associated with the current app's content. That container
object contains both private and public data as properties, so we'll assign our
publicDB and privateDB constants to the public and private data properties of our
container object. Notice how our class properties map well to the CloudKit dashboard. 

Next we want to implement a singleton pattern for our class. Singleton patterns are 
common in iOS and basically boil down to ensuring that every time you implement a 
class, you are referring to the exact same instance of that class rather than creating 
a new one. To do that, we assign a constant to an instance of our Model class. We
ensure to define our constant outside of the class declaration. Then we create a
type method that returns the same instance of the class every time its called. 

We're now going to create one last class in our models folder: Entry.

{x:entry_class}
Create a new file in your project navigator by going to file > new > file > 
swift file and name it "Entry.swift". Place this file in your Models folder. 

{x:entry_import_cloudkit}
In Models.swift, import Cloud Kit  and map kit right below the import Foundation statement in
the file. 

~~~language-swift
import Foundation
import CloudKit
import MapKit
~~~

Our class is going to get fairly large so let's start off simple and build it up.

{x:entry_properties}
Create a new class called Entry with the following properties:

~~~language-swift
class Entry: NSObject
{
  let model: Model = Model.singleton()
  let password: String
  var records = [Entry]()
  var record: CKRecord!
}
~~~

The first line assigns constant to the instance of our model class using the 
singleton() method. The second line creates a constant of type String that will
become a password/key to unlock our soon-to-be encrypted content. The records 
variable is an array of entries while the variable record is of type CKRecord.
CKRecord is a CloudKit dictionary object with key-value pairs that we use to 
fetch and save data.  

{x: initalize_entry}
Now add an initializer method to our class to to set the initial 
value of our properties. 


~~~language-swift
init(database: CKDatabase? = nil, record: CKRecord? = nil)
{
  self.record = (record != nil) ? record! : CKRecord(recordType: "Entry")
}  
~~~

Our initializer method is going to take a few optional properties as parameters. 
We create database and record params because we might want to pass in a specific 
database or record for development or testing purposes. We currently don't need
to do that so we are setting those optionals as nil. In the method itself we have a 
record property conditionally assigned to either the record parameter (if we've
pass one in) or to the "Entry" record type we defined in our Cloud Kit dashboard. 

Our initialization isn't actually done. We are going define a second initialization 
method called a convenience initializer. A convenience init is an initializer that
we use to make it easy to initialize a class for a particular use case. It differs
from the designated init() method in that it isn't guaranteed to be called like
designated initializers. You can instantiate a class with the designated 
initializer or the convenience init. The convenience init is really just a 
function because it is required to call the init() function. What makes it feel 
like an init() function is that we can call it like you would the 
init function. Let's write ours:

{x: initalize_entry}
Write a convenience initializer method for our Entry class with the following code:

~~~language-swift
convenience init(title: String, body: String, mood: Int, location: CLLocation?)
{
  self.init()
  
  self.title = title
  self.body = body
  self.mood = mood
  
  if let location = location
  {
    self.location = location
  }
}
~~~

In our convenience init we passing in title, body, mood, and location parameters and 
using those values to set the initial value of our self.title, self.body, and self.mood
properties. We also set the self.location property to the value of the location 
parameter, should it have a value. So when are we going to use our designated 
initializer and when are we doing to use our convenience init? We will use our
designated init when we need to instantiate a class but we don't need to set 
any initial values for our entry properties. An example of this would be if we need
to instantiate a class just to call a class method. We'd use a convenience init
in, for example, the EntryFormController to set the user inputs equal to the 
title, mood, body, and location properties of our Entry class before pushing
our data to the database. Look out for initializer calls in the rest of this 
tutorial and pay attention to how we decide to initialize our Entry class. 

{x: define_entry_property_vars}
Let's get rid of those compiler warnings by defining title, body, mood, and location
properties. 

~~~language-swift
var title: String {
  get {
    return (self.record!.objectForKey("Title") as! String
  }
  set (val) {
    record.setObject(val, forKey: "Title")
  }
}

var body: String {
  get {
    return (self.record!.objectForKey("Body") as! String
  }
  set (val) {
    record.setObject(val, forKey: "Body")
  }
}

var mood: Int! {
  get {
    return self.record!.objectForKey("Mood") as! Int
  }
  set (val) {
    record.setObject(val, forKey: "Mood")
  }
}

var location: CLLocation {
  get {
    return self.record!.objectForKey("Location") as! CLLocation
  }
  set (val) {
    record.setObject(val, forKey: "Location")
  }
}
~~~

Since our data is modeled directly after the CKRecord schema, we want our properties
to easily take values for any CKRecord we pull from our database and set property
values in our record (which itself is a CKRecord object) variable, which we can 
then save as a database entry. Our goal is to call Entry.title, for example, and
get record.title. When we set Entry.title, we want to also set record.title. 
The fastest way to do this is to declare our class properties as computed 
properties. Computed properties, as opposed to stored values 
because they don't actually store a value, instead they indirectly set the value
by using the setter and getter methods. 

Let's dive into the example of our title property. Title is an optional string
because we don't require users to enter a title for an entry. The get() method
will have us explicitly unwrap our record optional and use the ObjectForKey(key)
method to access the value for our "Title" key. <strong><i>Note: The key in the 
ObjectForKey(key) method is the attribute name in your Entry record table in 
the iCloud dashboard. The key is case sensitive so make sure to use the exact
name you used for the attribute name in the iCloud dashboard, lest you get "key 
not found" errors. </i></strong> Since we set the Title attribute to a string in
our iCloud dashboard, we can be sure the type cast to the String type will 
succeed, so we can use the forced form of the type cast operator (as!).

The setter method for our Title property takes a val parameter and calls the 
setObject(_:forKey:) method of the CLRecord class to assign a new value for the
key "Title." Our setters and getters for body, mood, and location follow the same
logic. 


## Encrypting Our Data

{video: flourish_encryption_intro}

Let's take a break for a minute from finishing up our Entry class to set up some
data encryption. Encryption? For a data app? Yes. Look we can be confident Apple
takes some care in securing iCloud data, but its best you do your own encryption 
for your app data in case of a breach. There are enough examples of security 
breaches at SaaS platforms to convince you to encrypt. This section will show
you that it actually isn't that hard to encrypt data by using one of the great
libraries out there for encryption. 

{x: create_library_folder}
Create a library folder by going to file > new > group and calling this folder 
"Library". This folder stores any external libraries in our project. 

{x: download_crypto_library}
Download our Encryptor library from [here](https://github.com/thinkclay/crypto) This is basically the 
great RNCryptor library found [here](https://github.com/RNCryptor/RNCryptor) with
some tweaks to work with Swift. 

{x: drag_crypto_library}
From your finder, drag the entire Encryptor folder into your Library folder to 
add it your project. 

The Encryptor library is written in Objective-C, so to expose those files to 
Swift, we need to add a file called a bridging header. 

{x: config_folder}
Create a config folder by going to file > new > group and calling this folder 
"Config". This folder will store our bridging header. 

{video: flourish_bridging_header}

{x: bridging_header}
Create a bridging header by going to file > new > file and selecting Header file.
Call this new file "Bridging-Header.h" 

{x: bridging_header_code}
In our bridging header we need to import the Crypto.h header file to expose it
to swift. Add the following to the Bridging-Header.h file:

~~~language-swift
#import "Crypto.h"
~~~

Now that we have our encryption library imported, we can start to use the it!
Our encryption library added a class we can call called Encryptor that has a few
simple methods that we will use in a custom class called AuthHelper. 

{video: flourish_auth_helper}

{x: helper_folder}
Create a helper folder by going to file > new > group and calling this folder 
"Helpers". This folder contains classes we deem to be helpers. That term isn't
a technical term, so just think of a helper as a class that we right to make our
lives a bit easier. 

{x: auth_helper_file}
Create a authentication helper file by going to file > new > file and selecting
Swift file. Call this new file "AuthHelper.swift" and add it to the Helpers 
folder.

Ok let's start to build out our AuthHelper class. 

{x: auth_helper_class}
Declare a new class called AuthHelper with the following code:


~~~language-swift
import UIKit

final public class AuthHelper: NSObject
{
  // silence is golden
}
~~~

This class declaration looks a bit different from what we've done so far. The final
keyword tells Swift that we won't be overriding it or its methods, so the system
can cache this class aggressively. We want our helper classes to be used as API,
so we don't want to restrict access to it using the public keyword during the 
declaration. Finally we are going to have our AuthHelper class as a subclass of 
NSObject.

{video: flourish_encrypt_method}

{x: password_constant}
In our AuthHelper class, let's define a class variable named named "key" and assign it a value of "testkey"

~~~language-swift
class var key:String {return "testKey" }
~~~

{x: entry_method}
Add an entry type method to our AuthHelper class with the following code:

~~~language-swift
public class func encrypt(input: String!, password: String? = nil) -> (string: String?, data: NSData?)
{
  let data = input.dataUsingEncoding(NSUTF8StringEncoding, allowLossyConversion: true)
  let pass = (password != nil) ? password : self.password
  let encryptedData = Encryptor.encryptData(data, password: pass, error: nil)
  let encryptedString = encryptedData.base64EncodedStringWithOptions(NSDataBase64EncodingOptions(rawValue: 0))
  
  return (string: encryptedString, data: encryptedData)
}
~~~

Our encrypt type method takes a an input string to be encrypted and a password 
string. It returns a tuple containing an encrypted string version of the data and
an NSData version of the data, both are optional because our encryption can 
fail. In this paradigm, we are using password as being synonymous with 
encryption key. 

The magic in our encrypt function happens by calling the encryptData() method of
our Encryptor class that takes a data param of type NSData, a password of type
NSString, and an error of type NSError. We are going to pass the error param as 
nil, though we do something with that callback in the future. Our password is
already the right type because in Swift a String type can be used pretty much 
interchangeably with NSString, which leaves us to deal only with our data param. 
Our data param is a String and it needs to be converted to an NSData type. 
Luckily, NSString has a method called  dataUsingEncoding(_:allowLossyConversion:) 
that allows returns an NSData object.In our data constant we call the 
dataUsingEncoding(_:allowLossyConversion:) of our input variable and choose to 
use UTF8 encoding and we set allowLossyConversion to true. 

The encryptData method encrypts our NSData object into an encrypted NSData object. 
Now we need to turn our encrypted NSData object into a String. The reason we do 
this is because strings are easy to store in databases and data objects are not. 
Therefore, we declare a constant called encryptedString that takes our
encryptedData object and uses base64EncodedStringWithOptions() method of the NSData
class to return an NSString version of our encrypted data. Finally we return the
tuple with both the string and data object versions of our encrypted data. 

Now we need to be able to decrypt data that has previously been encrypted. The 
logic for this method is going the inverse logic we used to in our encrypt function. 

{video: flourish_decrypt_method}

{x: decrypt_method}
Add an decrypt type method to our AuthHelper class with the following code:

~~~language-swift
public class func decrypt(input: String!, password: String? = nil) -> (string: String?, data: NSData?)
{
  let pass = (password != nil) ? password : self.password
  let encryptedData = NSData(base64EncodedString: input, options: NSDataBase64DecodingOptions(rawValue: 0))
  let decryptedData = Decryptor.decryptData(encryptedData, password: pass, error: nil)
  let decryptedText = (NSString(data: decryptedData, encoding: NSUTF8StringEncoding) as? String)
  
  return (string: decryptedText, data: decryptedData)
}
~~~

Now we need to hop back into our Entry.swift file so we can do a bit of refactoring
to show how simple it is to use our encryption methods. 

{video: flourish_refactor_entry}

{x: encrypt_entry_properties}
In Entry.swift, refactor your title and body properties to encrypt data in their
setter methods and decrypt data in their getter methods. 

Currently we have this:

~~~language-swift
var title: String {
  get {
      return (self.record!.objectForKey("Title") as! String)
  }
  set (val) {
      record.setObject(val, forKey: "Title")
  }
}

var body: String {
  get {
      return (self.record!.objectForKey("Body") as! String)
  }
  set (val) {
      record.setObject(val, forKey: "Body")
  }
}
~~~

Using our encrypt and decrypt methods looks like this: 

~~~language-swift
var title: String? {
  get {
    return AuthHelper.decrypt(self.record!.objectForKey("Title") as! String, password: password).string
  }
  set (val) {
    record.setObject(AuthHelper.encrypt(val, password: password).string, forKey: "Title")
  }
}

var body: String? {
  get {
    return AuthHelper.decrypt(self.record!.objectForKey("Body") as! String, password: password).string
  }
  set (val) {
    record.setObject(AuthHelper.encrypt(val, password: password).string, forKey: "Body")
  }
}
~~~

Notice we've changed our properties to optional String types. This is because
the encryption can fail so we might get a nil value from our getter method. We
are only encrypting the title and body properties of the entry in this app, but
feel free to encrypt as much or as little as you want.  

## CRUD Operations

{video: flourish_CRUD_create}

Now that we've set up our properties and learned about encrypting our data, we 
should jump into CRUD (Create Read Update Delete) methods in our Entry class. 

Let's begin with our create method: we set all of our property values in our 
initializer method so all we need to do is save our values to the database. That
means all we really need to know is whether or not the saving was successful, so
we should write a create method that takes a completion closure as a parameter. 

{x: create_method}
In Entry.swift, add a create method to the Entry class with the following code:

~~~language-swift
func create(completion: (success: Bool, message: String, error: NSError?) -> () )
{
  model.privateDB.saveRecord(record) { record, error in
    dispatch_async(dispatch_get_main_queue()) {
      if error != nil
      {
        completion(success: false, message: "Could not save to the cloud. Encrypted and saved locally instead.", error: error)
      }
      else
      {
        self.record = record
        completion(success: true, message: "Your entry was saved successfully!", error: nil)
      }
    }
  }
}
~~~

As promised, our create method takes a completion closure as the only parameter. 
The syntax order is first the name of our closure (here called "completion"),
then the parameter (a tuple with success, message, and error variables), 
followed by our return value (void). Our tuple parameter contains a success Bool
that will tell us if the saving succeeded or not, a message String we can use
to tell the user what's going, and an NSError that we can log or show the user. 

Inside the create method we use our model singleton's privateDB property. Remember
that privateDB is of time CKDatabase, which has a saveRecord() method. In our
code we call the method asynchronously, so the syntax might look confusing. Lets
take this one step at a time by going over saveRecord() and then making the call
asynchronous. The saveRecord() has the following format:   


~~~language-swift
func saveRecord(record: CKRecord!, completionHandler completionHandler: ((CKRecord!, NSError!) -> Void)!)
~~~

Save record takes a CKRecord parameter and a completion handler. The CKRecord we
pass into the method is what is going to be saved in our database and the completion 
handler will allow us to throw success and error messages in the callback. Here's
saveRecord using our variables:

~~~language-swift
func create(completion: (success: Bool, message: String, error: NSError?) -> () ) {
  model.privateDB.saveRecord(record, completionHandler: { record, error in
      if error != nil {
        completion(success: false, message: "Could not save to the cloud. Encrypted and saved locally instead.", error: error)
      }
      else {
        self.record = record
        completion(success: true, message: "Your entry was saved successfully!", error: nil)
      }
  })
}
~~~

If we have an error, our create method's completion closure returns a tuple with 
a message and an error we can display to the user. If we succeed in saving our record, 
we return a success tuple with a nil error and a message to indicate we've 
saved the data. 

{video: flourish_CRUD_create_async}

Now we can make our call to the saveRecord method asyncronous. The reason we do 
that is we don't want to hold up the execution of the rest of our code waiting 
for the completion closure to execute. We have no idea how long it is going to 
take to get a response from iCloud as to whether or not our save attempt was 
successful. In order to make execution of our code asyncronous, we use the dispatch_async
class of Apple's Grand Central Dispatch, which adds a block to a dispatch queue 
and allows the rest of our thread to continue to execute. 

{x: GCD}
If you want to learn more about Grand Central Dispatch as a whole, [read the
documentation on it](https://developer.apple.com/library/prerelease/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/func/dispatch_async)

{x: async_rewrite}
Refactor your create method to use the dispatch_async_method which results in the
following code:

~~~language-swift
func create(completion: (success: Bool, message: String, error: NSError?) -> () )
{
  model.privateDB.saveRecord(record) { record, error in
    dispatch_async(dispatch_get_main_queue()) {
      if error != nil
      {
        completion(success: false, message: "Could not save to the cloud. Encrypted and saved locally instead.", error: error)
      }
      else
      {
        self.record = record
        completion(success: true, message: "Your entry was saved successfully!", error: nil)
      }
    }
  }
}
~~~


{x: load_method}
In Entry.swift, add a load method to the Entry class with the following code:

~~~language-swift
func load(predicate: NSPredicate? = nil, sort: NSSortDescriptor? = nil)
{
  let predicate = predicate ?? NSPredicate(value: true)
  let sort = sort ?? NSSortDescriptor(key: "creationDate", ascending: false)
  let query = CKQuery(recordType: "Entry", predicate: predicate)
  query.sortDescriptors = [sort]
  
  model.privateDB.performQuery(query, inZoneWithID: nil) { results, error in
    if error != nil
    {
      dispatch_async(dispatch_get_main_queue()) {
        self.model.delegate?.errorUpdating(error)
        println("error loading: \(error)")
        return
      }
    }
    else
    {
      self.records.removeAll(keepCapacity: true)
      
      for record in results
      {
        let entry = Entry(record: record as? CKRecord)
        self.records.append(entry)
      }
      
      dispatch_async(dispatch_get_main_queue()) {
        self.model.delegate?.modelUpdated()
        println("successfully loaded entries")
        return
      }
    }
  }
}
~~~

{video: flourish_CRUD_load_constants}

As you may have already guess, this is the read method in our CRUD. We are going
to query our database and retrieve results for our query to use in our app. Those
results for our query will be ordered based on a property we pass in.  The 
load function is going to take two parameters:  a predicate (fancy iOS jargon 
for query) and a sort. The predicate is of type NSPredicate, which allows us to 
define logical conditions used to restrain our search. The sort parameter is of
type NSSortDescriptor, which takes a property name and a bool for if we want
ascending order. Both params are optionals that default to nil. 

Inside our load function we set two constants using the ?? function. This handy
function takes an optional on the left side and, if it doesn't have a value, 
assigns it a default value specified on the right hand side. Therefore line 1
of our function reads: "let the constant named predicate equal the value of our
predicate parameter if that parameter has a value. If not, our constant equals
NSPredicate(value:true)" The value:true statement is a special iOS predicate to 
return all entries that are nonempty. In line two we use the same logic to assign
our sort constant the value of either the sort param or 
NSSortDescriptor(key: "creationDate", ascending: false). As you can see, we are
ordering our results by date in descending order. In line 3 we declare a constant
with a value of a CKQuery instance. CKQuery's init method takes a recordType and
a predicate which we set to our recordType "Entry" and we pass in our predicate 
constant as well. In line 4 we use the sortDescriptors() method of CKQuery to pass
an array of sort objects. We only have one object in the array but we could pass
more than one search object. 

{video: flourish_CRUD_load_query}

Now that our variables are set up we can call the performQuery() method of 
CKDatabase and pass in our query and a completion block. The performQuery() also
takes a inZoneWithID parameter to constrain our results even more. We won't get
into that right now so we just set it to nil. 

The completion block of performQuery() method should look familiar as it is 
similar to our logic in the saveRecord() method we used earlier. When we have an 
error, we are going to finally use our model delegate's errorUpdating() method
to pass the error to the delegate. Notice our delegate is optional because we 
can't be sure anything has been assigned to be the delegate. 

If we don't have an error we start by calling self.records.removeAll to get rid
of any data we currently have, which we no longer need because we are about to
receive fresh, new data from the database. The keep capacity parameter tells iOS
to ensure the memory allotment can hold the given number of elements, even though
the array is now empty. Think of it as a way for us to tell iOS the array we just
emptied won't be empty for long. Next we have our for loop to iterate through 
our results. In each loop we essentially wrap our result which of type CKRecord 
in our Entry class so it can take advantage of all the great features in our 
custom class. After that we append that record into our records array. 

Finally, after our for loop we call the modelUpdated() method of model delegate 
and log a success message! Ok almost there with CRUD. 

Because we query the creationDate metadata of our record in our load() method, 
we must enable the sort and query options in our CloudKit dashboard.

{x: metadata_date_created} 
In your Cloud Kit dashboard, check the sort and query checkboxes for the 
Date Created attribute of your Entry record type. 

![query_sort_enable](/tuts_images/enable_query_sort.png)

Now that that's set, on to the update method!

{x: update_method}
In Entry.swift, add an update method to the Entry class with the following code:

~~~language-swift
func update(title: String? = nil, body: String? = nil, mood: Int? = nil, completion: (success: Bool, message: String, error: NSError?) -> () )
{
  self.title = title ?? self.title
  self.body = body ?? self.body
  self.mood = mood ?? self.mood
  
  create() { success, message, error in
    completion(success: success, message: message, error: error)
  }
}
~~~


{video: flourish_CRUD_update}

The update method is very similar to the create method. In fact, all we are
doing is calling our create method inside of our update method. Before we do that
we use the handy ?? again to see if any of our optional parameters as a value 
and assigning that value to our class properties. 


{video: flourish_CRUD_destroy}

{x: destroy_method}
In Entry.swift, add a destroy method to the Entry class with the following code:

~~~language-swift
func destroy(completion: (success: Bool, message: String, error: NSError?) -> () )
{
  model.privateDB.deleteRecordWithID(record.recordID) { record, error in
    if error != nil
    {
      println("ERR RECORD ID: \(self.record.recordID)")
      completion(success: false, message: "could not delete entry", error: error)
    }
    else
    {
      println("DEL RECORD ID: \(self.record.recordID)")
      completion(success: true, message: "successfully deleted entry", error: nil)
    }
  }
}
~~~

Destroy takes completion block and, as with our other create and load, we call
a class method of CKDatabase and do some error handling. This time we call
deleteRecordWithID(). We pass our record variable's recordID property and do our
error handling using the same logic we used in create where we return a tuple 
with a success Bool, a success/fail message, and an error if we have one. 
