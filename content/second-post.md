+++
title = "Using Async/Await and AsyncImage for SwiftUI app"
date = 2024-03-21

[taxonomies]
categories = ["Even"]
tags = ["SwiftUI", "Async/Await"]
+++

In this article, I will tell you about my first acquaintance with learning SwiftUI and using Swift Concurrency features.

<!-- more -->

I started learning SwiftUI with Apple’s Tutorial ["Introducing SwiftUI"](https://developer.apple.com/tutorials/swiftui).\
During the course, I was building the "Landmarks" app, which has a collection of famous nature places in the USA. This course reminded me of one of my first personal projects “Parks UK”. At that time I built my app using UIKit and it contains information about National Parks in the UK.

I decided that it would be good practice to rewrite my old app into SwiftUI using the same structure from the course. It wasn’t about just copying the code from the Apple course, as I faced some differences along the way and made some adjustments.

Let’s have a look at the differences and then we’ll dive into the details.

**Key differences between my project and Apple's project:**

1. **Data Parsing**: Apple's project parses JSON synchronously; I parse JSON asynchronously using `async/await`;
2. **Image Handling**: Apple's project has local storage for images in the `Assets` directory; I dynamically download images using `AsyncImage`;
3. **Preview Stability**: my use of Swift Concurrency caused preview crashes, but then I figured out how to fix it.

These differences are the key topics I will cover in this article.\
The link to my project "Park UK" is [here](https://github.com/marybokhan/parks-of-uk).

### Async/Await Usage

Let’s have a look at Apple’s "Landmarks" app function, which is responsible for getting the data from JSON:

```swift
func load<T: Decodable>(_ filename: String) -> T {
    let data: Data
    
    guard let file = Bundle.main.url(forResource: filename, withExtension: nil)
    else {
        fatalError("Couldn't find \(filename) in main bundle.")
    }
    
    do {
        data = try Data(contentsOf: file)
    } catch {
        fatalError("Couldn't load \(filename) from main bundle:\n\(error)")
    }
    
    do {
        let decoder = JSONDecoder()
        return try decoder.decode(T.self, from: data)
    } catch {
        fatalError("Couldn't parse \(filename) as \(T.self):\n\(error)")
    }
}
```

We can see that the data is parsed synchronously on the main thread. This means that UI can be blocked until we get all the data to display. It can work well with lightweight data stored locally, but what about JSON data we get from the server or with more massive data? More likely, the app will freeze waiting for the data to display. It can lead to a bad user experience. The solution is to parse JSON files asynchronously.

In my original "Parks UK" app, I used Grand Central Dispatch (GCD) for asynchronous loading. This time, I aimed to try a new `async/await` feature.

First, I created the function which parses JSON file:

```swift

func loadParks() async throws -> [Park] {
    guard let file = Bundle.main.url(forResource: "Parks", withExtension: "json")
    else {
        fatalError("Parks.json not found")
    }
    let data = try Data(contentsOf: file)
    let decoder = JSONDecoder()
    return try decoder.decode([Park].self, from: data)
}
```

I then asynchronously executed this function within a `Task`:

```swift
func load() {
    Task {
        do {
            self.parks = try await loadParks()
        } catch {
            fatalError("Error loading parks: \(error)")
        }
    }
}
```

And finally, triggered this loading process in the initializer:

```swift
init() {
    load()
}
```
This approach ensures data is loaded without blocking the UI, significantly improving app performance and user experience.

You can find these functions in my project “Parks UK” in the `ModelData` file.

### AsyncImage Usage

With the data structure ready and the JSON parsed, we're left to handle images, which are initially URL strings.

```swift
"imageURL": "https://www.nationalparks.uk/app/uploads/2020/09/hero-broads-600x400.jpg"
```

We need to convert this String into a URL and download the image from the URL, and only after that, we can display the image.

Previously, I managed image loading with GCD, but this time, I used SwiftUI's `AsyncImage` for a cleaner process.

In the `ParkRow` view, I integrated `AsyncImage` like so:

```swift
let imageURL = URL(string: park.imageURL)

AsyncImage(url: imageURL) { image in
	image.resizable()
} placeholder: {
	Image("park-placeholder").resizable()
}
.frame(width: 50, height: 50)
```

Now, the image is downloaded asynchronously and displayed in the app.

You may notice that I used a placeholder image. Using a placeholder image ensures users are not left staring at empty spaces during image loading, maintaining a well-designed user experience regardless of internet speed or link validity.

The placeholder image should be stored locally in the `Assets` directory.

Additionally, `AsyncImage` seamlessly integrates with custom views, as demonstrated in the `ParkDetail` view:

```swift
AsyncImage(url: URL(string: park.imageURL)) { image in
    CircleImage(image: image)
} placeholder: {
    CircleImage(image: Image("park-placeholder"))
}
.offset(y: -120)
.padding(.bottom, -100)
```

All set up! This method greatly simplifies the code for image loading. 

With an asynchronous approach, the app launches faster, and transitions become smoother. 

It could be the end of the story, but one more thing…

### Crashes in the Preview

After implementing an asynchronous approach in the app, I noticed that some Previews started to crash. This issue occurred specifically in files where I utilized JSON data. 

I found out that this happens due to the usage of Swift Concurrency.
This often happens because the Previews aren’t set up to handle asynchronous code correctly. SwiftUI Previews run synchronously, and starting asynchronous tasks directly in the view initializers or in the `onAppear` modifier can cause issues.

One of the solutions can be mock data for previews. Instead of performing actual asynchronous operations in previews (which cause crashes), we can use mock data. This approach will allow us to bypass the need for `async/await` in previews and still visualize the views with representable data. This approach is similar to writing mocks for tests.

Steps:

1. **Extend ModelData with a Mock Instance**
    
    First, let’s add an extension in the `ModelData` file with a static property called `mock`. Inside this property, we will manually create a `parks` mock.
    
    ```swift
    extension ModelData {
        static var mock: ModelData {
            let instance = ModelData()
            
            instance.parks = [
                Park(
                    id: 3,
                    name: "Exmoor",
                    country: "England",
                    isFavorite: false,
                    isFeatured: true,
                    imageURL: "https://...",
                    description: "...",
                    keyActivities: "Walking, ...",
                    website: "https://...",
                    coordinates: Park.Coordinates(...)
                ),
                Park(
                    ...
                ),
                ...
            ]
            return instance
        }
    }
    
    ```
    
    Inside the array, you can add as many mocks of the `Park` as you need.
    
    Here you can encounter an obstacle: you’re unable to create a `Park` instance, as there is no appropriate initializer available.
    This happens as the `Park` structure conforms to `Codable`, so it expects to be initialized automatically from a JSON by default. To create instances manually, we need to add a custom initializer to `Park`.

    The `Park` structure now looks like this:
    
    ```swift
    struct Park: Hashable, Codable, Identifiable {
        var id: Int
        var name: String
        var country: String
        var isFavorite: Bool
        var isFeatured: Bool
        var imageURL: String
        var description: String
        var keyActivities: String
        var website: String
        
        private var coordinates: Coordinates
        var locationCoordinate: CLLocationCoordinate2D {
            CLLocationCoordinate2D(
                latitude: coordinates.latitude,
                longitude: coordinates.longtitude
            )
        }
        
        struct Coordinates: Hashable, Codable {
            var latitude: Double
            var longtitude: Double
        }
        
        // Custom initializer for mock
        init(
            id: Int,
            name: String,
            country: String,
            isFavorite: Bool,
            isFeatured: Bool,
            imageURL: String,
            description: String,
            keyActivities: String,
            website: String,
            coordinates: Coordinates
        ) {
            self.id = id
            self.name = name
            self.country = country
            self.isFavorite = isFavorite
            self.isFeatured = isFeatured
            self.imageURL = imageURL
            self.description = description
            self.keyActivities = keyActivities
            self.website = website
            self.coordinates = coordinates
        }
    }
    
    ```
    
2. **Use Mock Data in Previews**
    
    Now we can use the mock data in places where a crash appeared. For example, let’s have a look at the `ParkDetail` file:
    
    ```swift
    #Preview {
        ParkDetail(park: ModelData.mock.parks[1])
            .environment(ModelData())
    }
    ```
    
    Voila! The crash disappeared, and now we can see the preview in the canvas.

### Lessons Learned

- Used the `async/await` for asynchronous parsing JSON data to improve the performance and user experience of the app;
- Used the `AsyncImage` to asynchronously download images from URL and simplify the code;
- Fixed crashes in SwiftUI Previews by adding mock data.
<br/><br/>

### Conclusion

Apple’s introductory course was a great start to getting familiar with SwiftUI. Using the same approach, I refactored one of my projects and learned how to use Swift Concurrency features and fix preview crashes.

I decided to document my learning experience due to the issues that appeared along the way. Despite reviewing numerous resources and forums, I couldn’t find clear answers to the problems. Ultimately, through trial and error and by consulting the documentation, I discovered the solutions.

I hope the topics covered in this article could be a great compliment to Apple’s introductory course.

Thank you for reading!