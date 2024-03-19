+++
title = "Using Async/Await and AsyncImage for SwiftUI app"
date = 2024-03-19

[taxonomies]
categories = ["Even"]
tags = ["SwiftUI", "Async/Await"]
+++

Hello! In this article I will tell you about my first acquaintance with learning Swift UI and using new swift concurrency features.

<!-- more -->

I started learning SwiftUI with Apple’s Tutorial “Introducing SwiftUI”. During the course we were building the Landmarks app, which have a collection of famous nature places in the USA. This course reminded me about one of my first personal projects “Parks UK”. That time I build my app using UIKit and it’s contains information about National Parks in the UK.

I decided that it will be a good practice to rewrite my old app into SwiftUI using the same structure from the course. It wasn’t about just copy the code from the Apple course, as I faced some differences along the way and made some adjustments.

Let’s have a look at the differences and then we’ll dive into the details.

**Key differences between my project and Apple project:**

1. they parse JSON synchronously; I parse JSON asynchronously using `async/await`;
2. they store all the images locally in the `Assets` directory; I download images dynamically from the Internet using `AsyncImage`;
3. they don’t have crashes with previews; I had crashes with previews as I used Swift concurrency, but then I figured out how to fix it.

So, these key difference are the key topics I will cover in this article.

### Async/Await Usage

Let’s have a look at Apple’s Landmarks app function, with is responsible for getting the data from JSON:

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

We can see that the data is parsed synchronously on the main thread. This means that UI can be blocked until we get all the data to display. It can work well with lightweight data stored locally, but what about JSON data that we get from server or with more massive data? More likely the app will freeze waiting for the data to display. It’s a bad user experience. We should parse JSON files asynchronously.