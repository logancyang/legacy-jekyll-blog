---
layout: post
title: "UIKit Fundamentals II - Table View"
comments: True
---

- `UITableView` has 2 delegate protocols, a total of 44 methods:
  - `UITableViewDelegate`: response to user **events**
  - `UITableViewDataSource`: access to **data** and cells

- `UITableViewDelegate`
  - What should happen when a button in a cell is tapped?
  - What should be the response to cell selection? (*)
  - How should I respond when a user begins editing a row?
  - What should happen when a cell is deselected?

- `UITableViewDataSource`
  - How many rows do I have? (*)
  - How many sections do I have?
  - What are the titles for the sections?
  - What is the cell view for each row? (*)

(*) marked points are the most essential when building a table view.

- TableView delegate method signatures: the first parameter of the methods in the TableView delegate
is always the TableView itself (similar to the TextField delegate introduced earlier).

{% highlight swift %}
// UITextFieldDelegate
func textField(textField: UITextField, shouldChangeCharactersInRange range: NSRange, \
                replacementString string: String) -> Bool {
            }

// UITableViewDelegate
func tableView(tableView: UITableView, canEditRowAtIndexPath indexPath: NSIndexPath) -> Bool {
            }
{% endhighlight %}

- `indexPath` is a struct with 2 members: row, section
  - For more on [structs](https://developer.apple.com/library/mac/documentation/Swift/Conceptual/Swift_Programming_Language/ClassesAndStructures.html#//apple_ref/doc/uid/TP40014097-CH13-ID83)

Some questions that can be answered by the methods:

1. How many rows? *Data Source Protocol method*

{% highlight swift %}
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int
{% endhighlight %}

2. What is the cell for each row? *Data Source Protocol method*

{% highlight swift %}
func cellForRowAtIndexPath(_ indexPath: NSIndexPath) -> UITableViewCell?
{% endhighlight %}

3. What should happen when a cell is selected?

{% highlight swift %}
func tableView(_ tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath)
{% endhighlight %}

### Practice App: FavoriteThings

(In `ios-nanodegree-uikit-2.0`, check out branch: `step4.1-favoriteThings-incomplete`)

The storyboard is set, write the view controller code.

- The correct implementation of getting the number of rows: use `numberOfRowsInSection` 
given by the DataSource protocol.

{% highlight swift %}
    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.favoriteThings.count
    }
{% endhighlight %}

- How `cellForRowAtIndexPath` Wrangles Cells
  - Cache the objects that are reused, only update the data. It lets scrolling become faster.

Steps for populating table cells:
(in `cellForRowAtIndexPath`...)

1. Dequeue a reusable cell (using the correct reuse identifier).
2. Find the model object for the corresponding row.
3. Set the images and labels in the cell
4. Return the cell

We use the standard `UITableViewCell` with property `textLabel`. 

Our reuse identifier is `FavoriteThingCell`.

{% highlight swift %}
    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
      
      // TODO: Implement method
      // 1. Dequeue a reusable cell from the table, using the correct “reuse identifier”
      // 2. Find the model object that corresponds to that row
      // 3. Set the images and labels in the cell with the data from the model object
      // 4. return the cell.
        
        let cell = tableView.dequeueReusableCellWithIdentifier("FavoriteThingCell")!
            as UITableViewCell
        cell.textLabel?.text = favoriteThings[indexPath.row]
        return cell
    }
{% endhighlight %}

Note: In `let cell...`, `as!` results in error, move `!` to the place it is to unwrap the optional:

`func dequeueReusableCellWithIdentifier(_ identifier: String) -> UITableViewCell?`

### Practice App: DoReMi:

The table in the storyboard is set, write the view controller code.

{% highlight swift %}
import UIKit

class ViewController: UIViewController, UITableViewDataSource {

    // Use this string property as your reuse identifier, 
    // Storyboard has been set up for you using this String.
    let cellReuseIdentifier = "MyCellReuseIdentifier"
    
    // Choose some data to show in your table
    // a list of dicts
    let model = [
        ["text" : "Do", "detail" : "A deer. A female deer."],
        ["text" : "Re", "detail" : "A drop of golden sun."],
        ["text" : "Mi", "detail" : "A name, I call myself."],
        ["text" : "Fa", "detail" : "A long long way to run."],
        ["text" : "So", "detail" : "A needle pulling thread."],
        ["text" : "La", "detail" : "A note to follow So."],
        ["text" : "Ti", "detail" : "A drink with jam and bread."]
    ]
    
    // Add the two essential table data source methods here
    
    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.model.count;
    }
    
    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        // bug: not as!
        let cell =  tableView.dequeueReusableCellWithIdentifier(self.cellReuseIdentifier)! as UITableViewCell
        
        let dictionary = self.model[indexPath.row]
        
        cell.textLabel?.text = dictionary["text"]
        cell.detailTextLabel?.text = dictionary["detail"]
        
        return cell
    }

}
{% endhighlight %}

### Practice App: Bond Villains

Written model and view controller code, set the table in the storyboard.

1. The Villain Struct
  - What are its properties?
  - Does it have any custom init methods?
  - How will the static helper methods allow users to construct the model for the app?

{% highlight swift %}
struct Villain {

    let name: String
    let evilScheme: String
    let imageName: String

    static let NameKey = "NameKey"
    static let EvilSchemeKey = "EvilScheme"
    static let ImageNameKey = "ImageNameKey"

    // Generate a Villain from a three entry dictionary

    init(dictionary: [String : String]) {
        self.name = dictionary[Villain.NameKey]!
        self.evilScheme = dictionary[Villain.EvilSchemeKey]!
        self.imageName = dictionary[Villain.ImageNameKey]!
    }
}
{% endhighlight %}

2. The XCAssets. How are the images tied to the data in the model?

3. The View Controller
  - How does the view controller get access to the model?
  - How are the UITableViewDataSource methods implemented?
  - What is the reuse identifier for the cells in this project? (This will be important in the next step)

{% highlight swift %}
import UIKit

class ViewController: UIViewController, UITableViewDataSource, UITableViewDelegate {

    // Get ahold of some villains, for the table
    // This is an array of Villain instances
    let allVillains = Villain.allVillains

    // MARK: Table View Data Source

    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return allVillains.count
    }

    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCellWithIdentifier("VillainCell")! as UITableViewCell
        let villain = allVillains[indexPath.row]

        // Set the name and image
        cell.textLabel?.text = villain.name
        cell.imageView?.image = UIImage(named: villain.imageName)

        // If the cell has a detail label, we will put the evil scheme in.
        if let detailTextLabel = cell.detailTextLabel {
            detailTextLabel.text = "Scheme: \(villain.evilScheme)"
        }

        return cell
    }

    func tableView(tableView: UITableView, canEditRowAtIndexPath indexPath: NSIndexPath) -> Bool {
        return true
    }

}
{% endhighlight %}

To setup the tableView in the storyboard:

1. Add a table to the main view
  - Drag a table view to the main view and leave a bit space above (just below the battery sign)
2. Add constraints: *add all missing constraints*
3. Connect the table's data source property to the view controller
  - Right click the table view, drag the small circle beside the outlet *dataSource* to the ViewController
  icon on top
4. Create a prototype cell
  - Go to Attributes inspector, set the *Prototype Cells* to 1
5. Set the cell's reuse identifier
  - Open Outline, under *Table View*, click *Table View Cell*
  - In the Attributes inspector, specify the reuse identifier as "VillainCell", the same as in the code.
  - HIT RETURN after entering the identifier! (ALWAYS hit return after inputting text in the inspectors,
  or the storyboard won't save it)

Running the app at this point, we can see the pictures and the names, but not the schemes.

Change the style for the cells to show details

### Challenge App: Roshambo With History

Begin the project by checking out step4.7-roshamboWithHistory-incomplete.





