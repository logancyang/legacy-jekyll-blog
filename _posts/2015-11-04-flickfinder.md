---
layout: post
title: "iOS Networking with Swift II: Flick Finder"
comments: True
---

With the knowledge of the [previous post](http://loganyc1934.github.io/2015/11/01/iOSNetworking/),
we can write a photo searching app with the Flickr API. In the Flickr universe, a photo is called
a _Flick_, so let's call this app _Flick Finder_. The project is 
[here](https://github.com/loganyc1934/FlickFinder).

### Concepts

- Built networking app step-by-step
- Constructed and called web service method
- Parsed JSON data from HTTP response
- Populated view using data from HTTP response
- Read an API's documentation to solve an issue: maximum number of photos returned, randomly choose
a page and an image.

**Get the NSURLSessions singleton**

```swift
let session = NSURLSession.sharedSession()
```

**Create and run a NSURLSessionDataTask**

```swift
let url = NSURL(string: "https://api.flickr.com/services/rest/
    ?method=flickr.photos.search&api_key=d72c5a85006014ea74022c115e4ebd5b&text=test&format=json
    &nojsoncallback=1&auth_token=72157650613647678-32c4dc1af8b80f31
    &api_sig=0353bdf97603872c2c2338390da3793d")!

let request = NSURLRequest(URL: url)

let task = session.dataTaskWithRequest(request) { data, response, downloadError in
    // do something here...
}

task.resume()
```

**Parse raw JSON data into NSDictionary (if response is a dictionary)**

```swift
var parsingError: NSError? = nil
let parsedResult = NSJSONSerialization.JSONObjectWithData(data, 
    options: NSJSONReadingOptions.AllowFragments, error: &parsingError) 
    as NSDictionary
```

**Shifting keyboard review**

Pay attention to the process of hiding the keyboard through both `tapRecognizer: UITapGestureRecognizer`
and the textField delegate. The components are:

- `tapRecognizer = UITapGestureRecognizer(target: self, 
    action: "handleSingleTap:")`

```swift
    func handleSingleTap(recognizer: UITapGestureRecognizer){
        view.endEditing(true)
    }
```

- `tapRecognizer?.numberOfTapsRequired = 1`
- `someTextField.delegate = self`

```swift
    // textField delegate
    func textFieldShouldReturn(textField: UITextField) -> Bool {
        textField.resignFirstResponder()
        return true;
    }
```

- `func addKeyboardDismissRecognizer()`, `func removeKeyboardDismissRecognizer()`

```swift
    func addKeyboardDismissRecognizer(){
        view.addGestureRecognizer(tapRecognizer!)
    }
    
    func removeKeyboardDismissRecognizer(){
        view.removeGestureRecognizer(tapRecognizer!)
    }
```

- `func subscribeToKeyboardNotifications`, `func unsubscribeToKeyboardNotifications`

```swift
    func subscribeToKeyboardNotifications(){
        NSNotificationCenter.defaultCenter().addObserver(self, 
            selector: "keyboardWillShow:", 
            name: UIKeyboardWillShowNotification, object: nil)
        NSNotificationCenter.defaultCenter().addObserver(self, 
            selector: "keyboardWillHide:", 
            name: UIKeyboardWillHideNotification, object: nil)
    }

    func unsubscribeToKeyboardNotifications(){
        NSNotificationCenter.defaultCenter().removeObserver(self, 
            name: UIKeyboardWillShowNotification, object: nil)
    }
```

- `func keyboardWillShow()`, `func keyboardWillHide()`, shift the keyboard

```swift
    /// when the keyboardWillShow notification is received, shift the view's frame up
    func keyboardWillShow(notification: NSNotification) {
        view.frame.origin.y = -getKeyboardHeight(notification)
    }
    
    /// when the keyboardWillHide notification is received, shift the view's frame down
    func keyboardWillHide(notification: NSNotification) {
        view.frame.origin.y = 0
    }

    func getKeyboardHeight(notification: NSNotification) -> CGFloat {
        let userInfo = notification.userInfo
        let keyboardSize = userInfo![UIKeyboardFrameEndUserInfoKey] 
            as! NSValue // of CGRect
        return keyboardSize.CGRectValue().height
    }
```

