# // MARK: - What is it? | Sarunw

If you have ever created a new view controller, you might have a chance to see this boilerplate code in the newly created view controller.

```swift
/*
// MARK: - Navigation

// In a storyboard-based application, you will often want to do a little preparation before navigation
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    // Get the new view controller using segue.destination.
    // Pass the selected object to the new view controller.
}
*/
```

If you are coding long enough, you might already know what it is. But for a newcomer, you might wonder what is `// MARK: - Navigation` doing here.

You can easily support sarunw.com by checking out this sponsor.

![MARK%20-%20What%20is%20it%20Sarunw%20959edbaea6b84a77bae37b873f9684e3/codeshot.png](MARK%20-%20What%20is%20it%20Sarunw%20959edbaea6b84a77bae37b873f9684e3/codeshot.png)

## What is this // MARK

`MARK` is one of a code annotation[[1]](https://sarunw.com/posts/mark-what-is-it/#fn1) that adds a visual clue to the jump bar[[2]](https://sarunw.com/posts/mark-what-is-it/#fn2) and minimap[[3]](https://sarunw.com/posts/mark-what-is-it/#fn3).

The following is an example of how `// MARK` shows up in the jump bar and minimap.

```swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    ...

    // MARK: UISceneSession Lifecycle
    func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration {
        // Called when a new scene session is being created.
        // Use this method to select a configuration to create the new scene with.
        return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
    }

    ...
}
```

The above code will result in a header name **"UISceneSession Lifecycle"** showing up in jump bar and minimap.

## Variations

There are some variations of code annotation, I will briefly go through all of them.

### Add a heading

Insert a comment (`//`) with the prefix `MARK:` followed by your section heading.

```
// MARK: View life cycle
override func viewDidLoad() {
    super.viewDidLoad()

    // Do any additional setup after loading the view.
}
```

You can also use `#pragma mark` and `#pragma mark -` in Objective-C. Which is equivalent to `// MARK:` and `// MARK: -`.

```
#pragma mark View life cycle
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
}
```

### Add a separator line

Throughout the article, you might notice a `MARK` with and without a hyphen (`-`). This hyphen has a special meaning. If you put a hyphen before the comment part of an annotation, it will add a separator above your annotation in three places: code, jump bar, and minimap.

```
// MARK: - Navigation
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    // Get the new view controller using segue.destination.
    // Pass the selected object to the new view controller.
}
```

There is also a less less-known variation on the separator. If you put a hyphen after a comment, it will add a separator below the comment.

```
// MARK: Navigation -
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    // Get the new view controller using segue.destination.
    // Pass the selected object to the new view controller.
}
```

If you want just a separator, that also possible with a hyphen without any comment.

```
// MARK: -
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    // Get the new view controller using segue.destination.
    // Pass the selected object to the new view controller.
}
```

There are two more annotations that don't have a visual clue on code or the minimap, but convey special meaning and icon in the jump bar.

### Add a to-do item

There is a `// TODO:` which will show as a to-do icon in the jump bar.

```
private func longProcessingTimeFunction() {
    // TODO: Need optimization
}
```

### Add a bug fix reminder

To add a bug fix reminder, use `FIXME`. Which will add a shiny band-aid icon in the jump bar. This one is quite standout from other annotations, which is good.

```
private func absolute(_ x: Int) -> Int {
    // FIXME: Returns incorrect values for negative arguments
    return x
}
```

You can easily support sarunw.com by checking out this sponsor.

[Turn your code into a snapshot: Codeshot creates a beautiful image of your code snippets. Perfect size for Twitter.](https://codeshotapp.com/)

## How to use

These annotations exist to help you annotate your code. So that you can section and quickly navigation within the file (MARK), pinpoint the remaining tasks (TODO), and remind you for the remaining problems (FIXME).

There is no rule on how to use these, this article shows you all possible ways to annotate your code. How do you use this tool is solely up to you.

Feel free to [tweet](https://twitter.com/sarunw) me your creative way of using these annotations. Right now, I mostly use it for section delegate and data source.

You may also like

[Multi-cursor editing in Xcode](https://sarunw.com/posts/multi-cursor-editing-in-xcode/)

It is a hidden gem in Xcode that can save up your coding time. Learn what it is, how to use it, and some use cases.

[Xcode](https://sarunw.com/tags/Xcode/) [Workflow](https://sarunw.com/tags/Workflow/)

[How to create code snippets in Xcode](https://sarunw.com/posts/how-to-create-code-snippets-in-xcode/)

Create a reusable boilerplate snippet that you can use in the project.

[Xcode](https://sarunw.com/tags/Xcode/) [Workflow](https://sarunw.com/tags/Workflow/)

[How to create a macOS app without storyboard or xib files](https://sarunw.com/posts/how-to-create-macos-app-without-storyboard/)

macOS is tightly coupled with storyboard and xib than iOS. To build your UI entirely in code, we have to do some initial setup.

[Xcode](https://sarunw.com/tags/Xcode/) [Workflow](https://sarunw.com/tags/Workflow/)

Read more article about [Xcode](https://sarunw.com/tags/Xcode/), [Workflow](https://sarunw.com/tags/Workflow/), or see [all available topic](https://sarunw.com/tags)

## Enjoy the read?

If you enjoy this article, you can subscribe to the weekly newsletter.**Every Friday**, you'll get a quick **recap of all articles and tips posted on this site**. No strings attached. Unsubscribe anytime.

Feel free to follow me on [Twitter](https://twitter.com/sarunw) and ask your questions related to this post. Thanks for reading and see you next time.

If you enjoy my writing, please check out my Patreon [https://www.patreon.com/sarunw](https://www.patreon.com/sarunw) and become my supporter. Sharing the article is also greatly appreciated.