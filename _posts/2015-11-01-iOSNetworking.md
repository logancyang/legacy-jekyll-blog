---
layout: post
title: "iOS Networking with Swift I: Components"
comments: True
---

Without networking, mobile apps can only perceive, process, and present the data that is local to 
the host device. This greatly restricts the scope of problems which apps can solve and limits their 
overall utility. By incorporating networking, apps truly become “mobile” -- they can interact with 
interesting data using popular web services, coordinate multi-user activities, and build experiences 
that bring users together.

This series of posts will cover concepts fundamental to communication over the network like HTTP, JSON, 
and authentication. These concepts are also highly transferrable to other platforms, languages, 
and applications. Additionally, it will mold your understanding of app design, especially when 
networking constraints are involved.


### Review: "Sleeping in the Library"

Here we use the [Flickr API](https://www.flickr.com/services/api/) 
as an example to show how to do networking with Swift.

#### 1. Define constants

Here, we define some constants used in the app. Each of these constants are used to construct the 
URL of the HTTP GET request.

```swift
/* 1 - Define constants */
let BASE_URL = "https://api.flickr.com/services/rest/"
let METHOD_NAME = "flickr.galleries.getPhotos"
let API_KEY = "ENTER_YOUR_API_KEY_HERE"
let GALLERY_ID = "5704-72157622566655097"
let EXTRAS = "url_m"
let DATA_FORMAT = "json"
let NO_JSON_CALLBACK = "1"
It makes sense to keep all of these as separate constants so that you can mix and match them to create various URLs later on (if we were to expand this app). It also makes sense to keep all these constants in one area, so if any of these bits that we’re getting from an external source (flickr) happen to change, we know where to go to make the necessary changes in our own app.
```

#### 2. API method arguments 

For convenience, we place the arguments used for the flickr.galleries.getPhotos method into a 
dictionary. This dictionary will be used by the escapedParameters() helper function to help 
construct the urlString.

```swift
/* 2 - API method arguments */
let methodArguments = [
    "method": METHOD_NAME,
    "api_key": API_KEY,
    "gallery_id": GALLERY_ID,
    "extras": EXTRAS,
    "format": DATA_FORMAT,
    "nojsoncallback": NO_JSON_CALLBACK
]
```

#### 3. Initialize session and url 

Here's where the good stuff starts. We'll explain this part more in detail later, but here are the essentials: 

`NSURLSession` — a class that provides an API for downloading content via HTTP. 

`sharedSession()` — a method of `NSURLSession`'s that returns what's called a "singleton instance" of `NSURLSession`. 

singleton — a design pattern that restricts the instantiation of a class to one object. Calling 
`NSURLSession.sharedSession()` returns a singleton instance of `NSURLSession`, the only one that can exist.

We now create the `urlString` and subsequently the `NSURL` object. Note that the `methodArguments` is
contained in the `urlString`, so the API call is completed by adding the API arguments in the URL, which
we use to initialize a request.

Lastly, we use the `NSURL` object to create an instance of `NSURLRequest`.

```swift
/* 3 - Initialize session and url */
let session = NSURLSession.sharedSession()
let urlString = BASE_URL + escapedParameters(methodArguments)
let url = NSURL(string: urlString)!
let request = NSURLRequest(URL: url)
```

#### 4. Initialize task for getting data

Using the session and the request, we instantiate what's called a **task** for grabbing an image from 
Flickr. We use the closure that starts out `{ data, response, downloadError in` as a completion handler. 
We will talk more about these later. Underneath that line, within the completion handler, 
is where we will work with the response data.

```swift
/* 4 - Initialize task for getting data */
let task = session.dataTaskWithRequest(request) { data, response, downloadError in
// handle the response...
})
```

#### 5. Check for a successful response 

In the completion handler, we use `guard` statements to check for a successful response to our request.

```swift
/* 5 - Check for a successful response */
/* GUARD: Was there an error? */
guard (error == nil) else {
    print("There was an error with your request: \(error)")
    return
}

/* GUARD: Did we get a successful 2XX response? */
guard let statusCode = (response as? NSHTTPURLResponse)?.statusCode where statusCode >= 200 && statusCode <= 299 else {
    if let response = response as? NSHTTPURLResponse {
        print("Your request returned an invalid response! Status code: \(response.statusCode)!")
    } else if let response = response {
        print("Your request returned an invalid response! Response: \(response)!")
    } else {
        print("Your request returned an invalid response!")
    }
    return
}

/* GUARD: Was there any data returned? */
guard let data = data else {
    print("No data was returned by the request!")
    return
}
```

#### 6. Parse the data (i.e. convert the data to JSON and look for values!) 

If we've gotten to this point, then our request must have been successful! So now we parse the JSON 
response data into something we can use.

```swift
/* 6 - Parse the data (i.e. convert the data to JSON and look for values!) */
let parsedResult: AnyObject!
do {
    parsedResult = try NSJSONSerialization.JSONObjectWithData(data, options: .AllowFragments)
} catch {
    parsedResult = nil
    print("Could not parse the data as JSON: '\(data)'")
    return
}

/* GUARD: Did Flickr return an error (stat != ok)? */
guard let stat = parsedResult["stat"] as? String where stat == "ok" else {
    print("Flickr API returned an error. See error code and message in \(parsedResult)")
    return
}

/* GUARD: Are the "photos" and "photo" keys in our result? */
guard let photosDictionary = parsedResult["photos"] as? NSDictionary,
    photoArray = photosDictionary["photo"] as? [[String: AnyObject]] else {
    print("Cannot find keys 'photos' and 'photo' in \(parsedResult)")
    return
}
```

#### 7. Generate a random number, then select a random photo 

Our request will return JSON for multiple images in gallery 5704-72157622566655097, 
but we just want to grab one of those images, at random. We then save the selected image's title and URL.

```swift
/* 7 - Generate a random number, then select a random photo */
let randomPhotoIndex = Int(arc4random_uniform(UInt32(photoArray.count)))
let photoDictionary = photoArray[randomPhotoIndex] as [String: AnyObject]
let photoTitle = photoDictionary["title"] as? String /* non-fatal */

/* GUARD: Does our photo have a key for 'url_m'? */
guard let imageUrlString = photoDictionary["url_m"] as? String else {
    print("Cannot find key 'url_m' in \(photoDictionary)")
    return
}
```

#### 8. If an image exists at the url, set the image and title 

If there is data at the URL, then we update the photoImageView and photoTitle! The line 
`dispatch_async(dispatch_get_main_queue(), {` allows us to quickly update the User Interface, 
so the picture and accompanying text can be seen right away.

```swift
/* 8 - If an image exists at the url, set the image and title */
let imageURL = NSURL(string: imageUrlString)
if let imageData = NSData(contentsOfURL: imageURL!) {
    dispatch_async(dispatch_get_main_queue(), {
        self.photoImageView.image = UIImage(data: imageData)
        self.photoTitle.text = photoTitle ?? "(Untitled)"
    })
} else {
    print("Image does not exist at \(imageURL)")
}
```

#### 9. Resume (execute) the task 

`task.resume()` is the line that actually starts the execution of our request. Up to this point, 
all the previous steps just define what we should do when the request executes and completes.

```swift
/* 9 - Resume (execute) the task */
task.resume()
```

### Summary of the steps

1. Client sends a HTTP request to the server (Flickr API) using the method `flickr.galleries.getPhotos`.
2. Server responds with JSON containing information about the photos in gallery.
3. Client stores the JSON, then randomly picks the URL of an image in the gallery.
4. Client requests an image's data using the URL.
5. Client displays the image.
6. (If the button is pressed, go back to step 1)

