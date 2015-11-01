---
layout: post
title: "UIKit Fundamentals II - Navigation, Collection View, Tab View"
comments: True
---

### Practice App: Make Your Own Adventure (MYOA)

Follow the steps below to recreate the MYOA on your own.

**Step 1**: Create a new single view application

**Step 2**: Delete the single view controller (You can delete this view controller by selecting the 
yellow icon and then tapping the delete button)

**Step 3**: Drag a UINavigationController into the storyboard from the object library.

**Step 4**: Delete the automatically generated root view controller and drag in a new view 
controller. To reset the Navigation Controller’s Root View Controller property control click on the 
Navigation Controller’s round yellow icon and drag a line from the root view controller triggered 
segue to your new view controller.

**Step 5**: Add two buttons to the bottom half of the view controller. Make them the full width of 
the view to allow room for text

**Step 7**: Add a UITextView to the top half. You might want to change the font in the attribute 
inspector.

**Step 8**: Change the text in the textview and buttons to provide a start for your story. To add 
newline characters to your text view, enter the text in the attribute inspector, and type 
option-return for a newline. The display of the text in Storyboard might be different than in 
the running app. Run the app at this point to check.

<br>
<center><img src="/assets/images/uikitII/nav01.png" alt="Drawing" style="width: 200px;"></center>
<br>

**Step 9**: Copy and paste two new view controllers, to serve as the next story nodes. To copy 
an entire view controller scene, drag a rectangle around the whole view controller in Storyboard, 
then type command-c, command-v. The new view controller will be pasted directly on top of the old, 
so it will be difficult to see it until it is dragged off.

**Step 10**: Change the text in the new nodes to continue the story.

**Step 11**: Add Triggered Segues to the two buttons. Control click on each of the buttons, and 
drag a line from Triggered Segue → action to the new view controller.

<br>
<center><img src="/assets/images/uikitII/nav02.png" alt="Drawing" style="width: 600px;"></center>
<br>

**Step 12**: Run, and verify that the navigation works.

**Step 13**: Repeat, until the story is complete. Delete the buttons in the controllers representing 
endpoints of the story.

For a sample project, checkout the branch **step5.1-makeYourOwnAdventure**.

<br>
#### Add a Start Over Button

**Step 1**: Set the class of each of the story nodes to “ViewController”. You can set them by 
selecting the round yellow icon for a view controller, then setting the class in the Identity Inspector.

**Step 2**: Add the right bar button. In the viewDidLoad() method ViewController.swift add the 
following code:

```swift
self.navigationItem.rightBarButtonItem = UIBarButtonItem (
                 title: "Start Over", 
                 style: UIBarButtonItemStyle.Plain, 
                target: self, 
                action: "startOver")
```

**Step 3**: Add the startOver method. Use the following code:

```swift
func startOver() {
        if let navigationController = self.navigationController {
           navigationController.popToRootViewControllerAnimated(true)
        }
}
```

To see a solution checkout the branch **step5.2-makeYourOwnAdventure-startOver**.

**NavigationController has a stack of ViewControllers**.

Use `deinit` deinitializer to print out a message to see the stack behavior.

```swift
deinit {
    print("ViewController deallocated")
}
```

And we can see that **a ViewController is deinitialized when it slides off the screen to the right**.
That's when it's popped off the stack.

### Practice App: Bond Villain - Revisited

Presenting the DetailViewController,

- Add the tableView(tableView: UITableView, didSelectRowAtIndexPath) method to ViewController
- When a row is selected, push the detail view controller onto the navigation stack.

```swift
func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
    let storyboard = self.storyboard!
    let detailController = storyboard.instantiateViewControllerWithIdentifier
    ("VillainDetailViewController") as! VillainDetailViewController
    detailController.villain = self.allVillains[indexPath.row]
    self.navigationController!.pushViewController(detailController, animated: true)
}
```

For the completed code for Bond Villains with Master-Detail checkout **step5.6-bondVillains-masterDetail**.

### Practice App: Bond Villain - Collection View

There are two ways to set up table view controllers and collection view controllers in Storyboard:

1. Start with a UIViewController and drag in a TableView or CollectionView.
2. Use a TableViewControlller or CollectionViewController.

In the table views lesson we used option 1. Now we’ll set up our collection view using option 2.

For practice, go ahead and open up the BondVillainsInTabs project from this branch: 
**step6.3-bondVillains-noCollectionView**

Steps to add a CollectionViewController alongside the TableViewController:

- Drag in a Navigation Controller.
- Delete the root view controller that comes with the Navigation Controller.
- Drag in a CollectionViewController.
- Set the root view controller of the Navigation Controller to be the CollectionViewController you just added.
- Add the Collection View Controller to the tab bar. To do this, control-click on the Tab Bar Controller. Under Triggered Segues drag from viewControllers to the Navigation Controller you just added.
- Open the identity inspector and set the class of the CollectionViewController to “VillainCollectionViewController”. (For MemeMe, this will be the “SentMemesCollectionViewController”).
- Click on the Collection View Cell in your Collection View.
- Open the identity inspector and set the cell class and reuseIdentifier to be “VillainCollectionViewCell”. (For MemeMe, this will be the custom cell class you created earlier.)
- For practice, drag a UIImageView into the Collection View Cell.
- Connect the imageView outlet of the VillainCollectionViewCell to the UIImageView you just added.

Note: be careful with the cell's reuseIdentifier. I had a typo with the wrong case of the first letter and it crashed.

### Troubleshooting Collection Views

**"I can’t see my Sent Memes Collection View."**

Check to make sure the classes for the CollectionViewController and the MemeCollectionViewCell 
have been set in storyboard.

The name of the CollectionViewController class, for example “SentMemesCollectionViewController”, 
needs to be set in the identity inspector so that it matches that of the .swift file.

The name of your custom collection view cell, for example, “CustomMemeCell” needs to be entered in 
two places:as the class name in the identity inspector and as the reuseIdentifier in the 
attributes inspector.

**"I added a new meme, but I don’t see it in my Sent Memes Collection View."**

_Your memes array copy may not be up to date._

You can ensure that your memes array is updated by defining the memes property in your 
CollectionViewController so that it accesses the memes array stored in the AppDelegate, 
similar to what is shown in the code below.

```swift
var memes: [Meme]! {
    let object = UIApplication.sharedApplication().delegate
    let appDelegate = object as! AppDelegate
    return appDelegate.memes
}
```

Your collection view may not be loading the most recent version of the memes array.

If you are adding or deleting memes and you don’t see this reflected in your Sent Memes Collection 
View you may need to refresh the collection view. The simplest way to do this is to call the 
UICollectionView method, `reloadData()`. If `reloadData()` is called in the viewWillAppear method of 
your CollectionViewController, the collection view will be refreshed every time the view appears.

Calling `reloadData()` looks something like this: `self.collectionView!.reloadData()`

Note: For better performance, there are other more refined methods specifically for adding and 
deleting items from a collection. Read more about them 
[here](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UICollectionView_class/#//apple_ref/doc/uid/TP40012177-CH1-SW35).

### Challenge App: Data-Driven MYOA


### MemeMe 2.0 Note

- Put the bottom tool bar above the bottom tab bar (which is automatically added by the tap view controller)
to avoid being hidden.
- Must set row height in the size inspector for **BOTH** the table view and the cell to change cell height.
- Do not forget to set table view's delegate and dataSource (right click it in the outline) to the 
view controller in storyboard (the yellow icon).
- Do not declare new array and make copy in viewDidLoad(), instead, declare `memes` array like this
in the table view controller.

```swift
var memes: [Meme]{ return (UIApplication.sharedApplication().delegate as! AppDelegate).memes }
```

- Use **UICollectionViewFlowLayout** to customize the collection view.

```swift
@IBOutlet weak var flowLayout: UICollectionViewFlowLayout!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let space: CGFloat = 1.0
        let dimension = (view.frame.size.width - 2 * space) / 3.0
        
        flowLayout.minimumInteritemSpacing = space
        flowLayout.minimumLineSpacing = space
        flowLayout.itemSize = CGSizeMake(dimension, dimension)
    }
```







