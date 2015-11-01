---
layout: post
title: "UIKit fundamentals I - Actions & Outlets, Presenting ViewControllers, Delegates"
---

## Outlets and Actions


#### Actions vs Outlets

Outlet: ViewController talks to View by using Outlet. Any object (UILabel, UIButton, UIImage, UIView etc) 
in View can have an Outlet connection to ViewController. Outlet is used as @property in ViewController 
which means that:

- you can set something (like Update UILabel's text, Set background image of a UIView etc.) of an object 
by using outlet.
- you can get something from an object (like current value of UIStepper, current font size of a 
NSAttributedString etc.)

Action: View pass on messages about view to ViewController by using Action (Or in technical terms ViewController set itself as Target for any Action in View). Action is a Method in ViewController (unlike Outlet which is @property in ViewController). Whenever something (any Event) happens to an object (like UIbutton is tapped) then Action pass on message to ViewController. Action (or Action method) can do something after receiving the message.
Note: Action can be set only by UIControl's child object; means you can't set Action for UILabel, UIView etc.


#### Practice App: Color Maker

- To connect IBOutlet weak var with UISwitch (or other UI components),
drag the small circle next to the var to the UISwitch.
- To connect IBAction func to UISwitches, drag the small circle to one/all of them.
- Pitfall: pay attention to the constraint. If there are negative constraints, there might be something wrong. This time, I slid the slider outside the UIImageView that holds the sliders so it appeared to be stuck at the rightmost edge.


## Presenting View Controllers

- Navigation View vs. Modal View

Navigation View: a view that enables the user to choose among views, shows the connections between different views.

Modal View: a view in which a user focuses on one task.


#### Practice App: Image Picker, Activity ViewController

- To display the Apple Image Picker which enables users to pick images from camera roll, just add a button and its an action like this.

{% highlight swift %}
    @IBAction func presentImage(sender: AnyObject) {
        let nextController = UIImagePickerController()
        self.presentViewController(nextController, animated: true, completion: nil)
    }
{% endhighlight %}

- To display the Apple Activity ViewController which enables users to choose activities like saving and sharing documents, just add this action.

{% highlight swift %}
    @IBAction func presentActivity(sender: AnyObject) {
        let image = UIImage()
        let controller = UIActivityViewController(activityItems: [image],
            applicationActivities: nil)
        self.presentViewController(controller, animated: true, completion: nil)
    }
{% endhighlight %}


- To display the Apple Alert Controller, add this action.

{% highlight swift %}
    @IBAction func presentAlert(sender: AnyObject) {
        let controller = UIAlertController()
        // must add title and message to display
        controller.title = "Test alert"
        controller.message = "This is a test"
        // add a cancel button
        let okAction = UIAlertAction(title: "ok", style: UIAlertActionStyle.Default){
            action in self.dismissViewControllerAnimated(true, completion: nil)
        }
        controller.addAction(okAction)
        // present the alert view
        self.presentViewController(controller, animated: true, completion: nil)
    }
}
{% endhighlight %}

Note that the Alert View doesn’t have a built-in cancel button like the previous two. 
We add the UIAlertAction okAction to dismiss the Alert View.


#### Practice App: Dice

Three ways of performing a segue:

1. Display the Dice View Controller with only code: when we want a reference of a ViewController in 
an Action, we identify this ViewController via its storyboard ID (it’s in the Identity inspector) by 
using storyboard?.instantiateViewControllerWithIdentifier(“storyboardID”) method. This method returns 
a view controller and must be downcast to DiceViewController.

{% highlight swift %}
    @IBAction func rollTheDice() {
        var controller: DiceViewController
        // set reference to the storyboard "DiceViewController"
        // the referenced storyboard must have this ID
        controller = self.storyboard?.instantiateViewControllerWithIdentifier("DiceViewController") as! DiceViewController
        
        controller.firstValue = self.randomDiceValue()
        controller.secondValue = self.randomDiceValue()
        
        presentViewController(controller, animated: true, completion: nil)
    }
{% endhighlight %}

2. Display the Dice View Controller with code and storyboard: to use storyboard to set the segue, we first select the first view, and ctrl click drag to the second view. Select Modal in this case. Then select the segue, set the Identifier to rollDice in the Attributes inspector. 

{% highlight swift %}
    @IBAction func rollTheDice() {
        performSegueWithIdentifier("rollDice", sender: self)
    }
{% endhighlight %}

3. Display the Dice View Controller with only storyboard: we directly ctrl click drag from the UIButton to the new view, and set the Identifier of the segue in the Attributes inspector. Clear the action in it.

- To pass data between ViewControllers, we use the method prepareForSegue(). The key is to define the destination view controller so that we can represent it and use its properties, firstValue and secondValue in this case.

{% highlight swift %}
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
    // in complex apps, check which segue is being called
        if segue.identifier == "rollDice" {
            let controller = segue.destinationViewController as!
            DiceViewController
            controller.firstValue = self.randomDiceValue()
            controller.secondValue = self.randomDiceValue()
        }        
    }
{% endhighlight %}


#### Practice App: RockPaperScissors

Note:

1. After adding the new view controller and setting its storyboard identifier, I encountered a strange 
runtime error that said the UIViewController cannot downcast to my second ViewController. Some searching 
on Stackoverflow solved it. Just select the view controller and in the Identity inspector, under Module, 
hit enter or select the module.

2. For UIImageView, ctrl click drag on it can set the image height and width.

3. Question remained for navigation bar and back button. They are missing.

See Github.



## Delegates


#### Delegate: an object that executes a group of methods on behalf of another object.

The view needs some other class to deal with user input. A view’s delegate is usually a control object. 
Control objects are designed to perform tasks like passing user input to a data model. The delegate pattern 
is that the view establishes questions that need answers and encode them in a protocol. Protocol is a list 
of methods that delegate must implement. Any object that fulfills a protocol can become a delegate.


#### Analogies to Protocols in the Physical World

There are interesting ways that connections in real life mirror delegate protocols inside a program.

Notice how the items on the left implement a standard (protocol) and that allows them to receive 
something from the item on the right:

Human Ear. The sounds involved in speech can be received by any ear that is tuned to the frequency 
range of the speaker. 

Electric plug. Any object that implements a standard electric plug protocol can receive electricity 
from a corresponding socket. Note that different regions of the world implement the electric plug protocol 
in different ways, but they all share basic components.


#### Emoji vs. Colorizer Textfields

<img src="/assets/images/controlFlow.png" alt="Drawing" style="width: 1000px;"/>

The first 3 steps are behind the scene, the first step we can see is the 4th.

<img src="/assets/images/emojiColorizer.png" alt="Drawing" style="width: 1000px;"/>

Emoji textfield clears text when editing begins; colorizer does not.

Colorizer hides keyboard on pressing return; Emoji does not.

Neither called textFieldDidEndEditing(), it saves the data to the model.


#### Write a delegate file: RandomColorTextFieldDelegate.swift

- Click the delegate folder, add new file, swift file, name it.
- Import UIKit, define the class, subclassing NSObject, declare UITextFieldDelegate protocol.

{% highlight swift %}
import Foundation
import UIKit

class RandomColorTextFieldDelegate: NSObject, UITextFieldDelegate {
    
    let colors = [UIColor.redColor(),
        UIColor.orangeColor(), UIColor.yellowColor(),
        UIColor.greenColor(), UIColor.blueColor(),
        UIColor.purpleColor(), UIColor.brownColor()]
    
    func randomColor() -> UIColor {
        //TODO: return a random color
        let randomIndex = Int(arc4random()) % colors.count
        
        // We get a color from the colors array using the random number as an index
        return colors[randomIndex]
    }
    
    // The color of the text in the textField is set to the randomly chosen color.
    func textField(textField: UITextField, shouldChangeCharactersInRange range: NSRange, replacementString string: String) -> Bool {
        textField.textColor = self.randomColor()
        return true
    }
}
{% endhighlight %}


#### Practice App: textFields


#### App: MemeMe 1.0

- Do not change the name of a set outlet, it’s in the xib file. If name is changed, there will be an uncaught exception saying the key of that variable has a problem, though build will succeed but the app will crash. If we absolutely want to change the name, delete the link of the outlet and set a new link and a new outlet. To delete the link, open the Connection inspector.
- To have the original aspect ratio in the UIImageView, select Mode Aspect Fit instead of Scale to Fill in the Attributes inspector.
- To set the constraints as they are showing, use the small triangular button at the bottom right.
- Declare the delegates: UIImagePickerControllerDelegate, UINavigationControllerDelegate
- Using the delegate method:

{% highlight swift %}
    @IBAction func pickAnImage(sender: AnyObject) {
        let pickerController = UIImagePickerController()
        pickerController.delegate = self
        self.presentViewController(pickerController, animated: true, completion: nil)
    }
    
    // Image Picker Delegate
    
    func imagePickerController(picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [NSObject : AnyObject]) {
        if let image = info[UIImagePickerControllerOriginalImage] as? UIImage {
            imagePicker.image = image
            self.dismissViewControllerAnimated(true, completion: nil)
        }
    }
{% endhighlight %}

- The ordering of objects in the document outline matters, it’s the ordering of the layers in the layout. For example, objects ordered lower means it’s above the upper objects.
- To separate Bar Button Items on the toolbar, drag a Flexible Space Bar Button item between them.
- To set the UITextField background as transparent,

{% highlight swift %}
self.myTextField.backgroundColor = UIColor.clearColor()
{% endhighlight %}

- To avoid UITextField attributes getting overridden, set other attributes like textAlignment and background color after setting defaultTextAttributes.
- To hide the UITextField border, click it and choose in the Attributes inspector.
- To move the view up and down when the keyboard shows and hides, for the bottom UITextField, do this

{% highlight swift %}
    // when the keyboardWillShow notification is received, shift the view's frame up
    func keyboardWillShow(notification: NSNotification) {
        if bottomTextField.isFirstResponder() {
            self.view.frame.origin.y -= getKeyboardHeight(notification)
        }
    }
    
    // when the keyboardWillHide notification is received, shift the view's frame down
    func keyboardWillHide(notification: NSNotification) {
        if bottomTextField.isFirstResponder() {
            self.view.frame.origin.y += getKeyboardHeight(notification)
        }
    }
{% endhighlight %}

- To generate memed image, (hide the notification bar and toolbar)

{% highlight swift %}
TODO
{% endhighlight %}

- To
- To

