# How To Secure iOS User Data: Keychain Services and Biometrics with SwiftUI | Kodeco

Apple’s *Keychain Services* is a mechanism for storing small, sensitive data such as passwords, encryption keys or user tokens in a secure and protected manner. Using Keychain Services, you can check that the password your user is entering matches their stored password without putting data at risk. However, entering a password is tedious! To solve this problem, Apple added *biometric authentication* to many devices. Biometric authentication allows users to confirm their identity quickly and securely, using either a fingerprint or a face scan.

In this tutorial, you’ll learn how to integrate Keychain Services and biometric authentication into a simple password-protected note-taking SwiftUI app.

## Getting Started

Download the starter project by clicking the *Download Materials* button at the top or bottom of the tutorial.

Open the starter project. Build and run. On the first run, the app prompts you for a password to protect your note. Go ahead and type a password into the two fields and tap *Set Password*. You’ll see a simple note-taking app that allows you to enter text and return to it later.

*Note*: Don’t worry if you forget your password – when running debug builds there’s a magical button to reset the app back to its initial state. You’ll lose your secret note, but you won’t be stuck.

The app wraps a *UITextView* to manage the note. Above the editor, you’ll see three buttons. On the far left is a trash can button, which is the reset button mentioned above. On the right is one button for changing the password and another for locking and unlocking the note.

Since you just entered a new password, the note is now unlocked and ready for editing. Type some text into the text editor.

![How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/starter-app-1.png](How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/starter-app-1.png)

Now stop and restart the app. The app starts with the note locked. The editor window is blurred to obscure the contents. Tap the lock button and enter the password you set earlier to unlock your note. The app uses User Defaults to store your note and password.

This makes it easy for an attacker with physical access to a device to locate the password! Keychain Services provides a safer place to store the password. In the next section, you’ll begin to update the app to do this.

## A Look at Keychain Services

Keychain Services provides an encrypted database managed by the operating system. It is for storing passwords, cryptographic keys, certificates or any short piece of data.

To store something in your keychain, you package several attributes about the data along with the secret information. You store all of them together into a *keychain item*. Keychain Services provides classes for different types of items:

- `kSecClassInternetPassword` stores a password for an internet site.
- `kSecClassGenericPassword` stores a password of any type.
- `kSecClassCertificate` stores a certificate.
- `kSecClassKey` stores a cryptographic key item.
- `kSecClassIdentity` stores an identity.

Each class uses a different set of attributes. These attributes define the information needed to identify the secured item. They also control access to the secret information. You can use the attributes to search for the item at a later time. If needed, you can share a keychain among apps.

![How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/keychain-services.png](How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/keychain-services.png)

The Keychain Services API has been around a long time. That means it’s a proven, reliable and safe place to store information. Unfortunately, it also means you’re going to deal with an API written for C! :]

But don’t panic. You’re going to write wrapper functions to let you deal with the API in a more modern fashion.

## Enabling Your Keychain

Now it’s time to set up your keychain. You’ll enhance the app to add, retrieve, update, and delete a password… securely!

### Adding a Password to the Keychain

In the starter project, open *KeychainServices.swift* in the *Models* group. You’ll see the definition for `KeychainWrapperError`, a custom `Error` that you’ll use to provide feedback to the user.

The first thing you’ll add is an initial definition for `KeychainWrapper`. Insert the following code at the end of the file, after `KeychainWrapperError`:

```swift
class KeychainWrapper {
  func storeGenericPasswordFor(
    account: String,
    service: String,
    password: String
  ) throws {
    guard let passwordData = password.data(using: .utf8) else {
      print("Error converting value to data.")
      throw KeychainWrapperError(type: .badData)
    }
  }
}

```

You must first convert the password from a `String` to `Data`. If the conversion fails, you throw an error.

*Note*: As you add code, you’ll see warnings about constants being “defined but never used”:

Don’t worry — you’ll resolve those warnings later in the tutorial. You can ignore them for now.

Access to Keychain Services works through a *query*. The first step in accessing Keychain Services is to create an *add query*. As the name implies, the add query defines the data you wish to store in the keychain.

Add the following code to the end of `storeGenericPasswordFor(account:service:password:)`:

```swift
// 1
let query: [String: Any] = [
  // 2
  kSecClass as String: kSecClassGenericPassword,
  // 3
  kSecAttrAccount as String: account,
  // 4
  kSecAttrService as String: service,
  // 5
  kSecValueData as String: passwordData
]

```

Here’s what’s happening:

1. The query is a dictionary that maps a `String` to an `Any` object, depending on the attribute. This pattern is common when calling C-based APIs from Swift. For each attribute, you supply the defined global constant beginning with *kSec*. In each case, you cast the constant to a `String` (it’s a `CFString` really), and you follow it with the value for that attribute.
2. The fist key defines the class for this item as a generic password, using the pre-defined constant `kSecClassGenericPassword`.
3. For a generic password item, you provide an account, which is your username field. You passed this into the method as a parameter.
4. Next, you set the service for the password. This is an arbitrary string that should reflect the purpose of the password, for example, “user login”. You also passed this into the method as a parameter.
5. Finally, you set the data for the item using the `passwordData` converted from the string passed into the method.

Now that you’ve built the query, you’re ready to store the value. Add the following code after the query definition:

```swift
// 1
let status = SecItemAdd(query as CFDictionary, nil)
// 2
switch status {
// 3
case errSecSuccess:
  break
// 4
default:
  throw KeychainWrapperError(status: status, type: .servicesError)
}

```

Here’s what this code is doing:

1. `SecItemAdd(_:_:)` asks Keychain Services to add information to the keychain. You cast the query to the expected `CFDictionary` type. C APIs often use the return value to show the result of a function. Here the value has type `OSStatus`.
2. You switch on the various values of the status code. It might seem odd to use a switch where you’re only checking one value, but who knows what might happen in the future ;]
3. `errSecSuccess` means your password is now in the keychain. Your work here is done!
4. If `status` contains another value, the function failed. `KeychainWrapperError` includes an initializer which uses `SecCopyErrorMessageString(_:_:)` to create a human-readable message for the exception.

You use this same pattern to access all Keychain capabilities: First, you create a query defining the work to do, and then you call a function with that query.

You now have a method to store a password in the keychain. Next, you’ll add *search* functionality to find and retrieve the item you just added.

### Searching for Keychain Items

The steps to read an item from the keychain mirror those to add the item. Add the following new method to the end of the `KeychainWrapper` class:

```swift
func getGenericPasswordFor(account: String, service: String) throws -> String {
  let query: [String: Any] = [
    // 1
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: account,
    kSecAttrService as String: service,
    // 2
    kSecMatchLimit as String: kSecMatchLimitOne,
    // 3
    kSecReturnAttributes as String: true,
    kSecReturnData as String: true
  ]
}

```

Again, the first step when reading an item from the keychain is to create the appropriate query:

1. You supplied `kSecClass`, `kSecAttrAccount` and `kSecAttrService` when adding the password to the keychain. You can now use these values to find the item in the keychain.
2. You use `kSecMatchLimit` to tell Keychain Services you expect a single item as a search result.
3. The final two parameters in the dictionary tell Keychain Services to return all data and attributes for the found value.

After the query, add the following code:

```swift
var item: CFTypeRef?
let status = SecItemCopyMatching(query as CFDictionary, &item)
guard status != errSecItemNotFound else {
  throw KeychainWrapperError(type: .itemNotFound)
}
guard status == errSecSuccess else {
  throw KeychainWrapperError(status: status, type: .servicesError)
}

```

First, you define an optional `CFTypeRef` variable to hold the value Keychain Services will hopefully find. You then call `SecItemCopyMatching(_:_:)`. You provide the query and a pointer to the destination value. This function searches the keychain for a match and copies the match to `item`.

<aside>
💡 *Note*: This may be a pattern you’ve not seen before. The ampersand (“&”) before a parameter means it’s a pointer to a memory slot rather than the value itself. The C function will update the memory at that location with a new value.

</aside>

Again, the status code provides error information. This code contains a specific error when Keychain Services doesn’t find the requested item.

Now you have the keychain item, but as a `CFTypeRef`. Add the following code at the end of `getGenericPasswordFor(account:service:)`:

```swift
// 1
guard
  let existingItem = item as? [String: Any],
  // 2
  let valueData = existingItem[kSecValueData as String] as? Data,
  // 3
  let value = String(data: valueData, encoding: .utf8)
  else {
    // 4
    throw KeychainWrapperError(type: .unableToConvertToString)
}

// 5
return value

```

Here’s what’s happening:

1. Cast the returned `CFTypeRef` to a dictionary.
2. Extract the `kSecValueData` value in the dictionary and cast it to `Data`.
3. Attempt to convert the data back to a string, reversing what you did when storing the password.
4. If any of these steps return `nil`, this means the data couldn’t be read. You throw an error.
5. If the casts and conversions succeed, you return the string containing the stored password.

Now you’ve implemented all the capabilities needed to store and retrieve a keychain item. Next, you’ll hook up your user interface so you can see it work!

<aside>
💡 以上是通过C保存和获取Keychain的部份

</aside>

### Using Keychain Services in SwiftUI

Open *NoteData.swift* under the *Models* group. Access to the password from the app uses two methods: `getStoredPassword()` reads the password, and `updateStoredPassword(_:)` sets the password. There’s also a property, `isPasswordBlank`.

First, delete the statement `let passwordKey = "Password"` at the start of the class.

Now replace the existing `getStoredPassword()` method with this:

```swift
func getStoredPassword() -> String {
  let kcw = KeychainWrapper()
  if let password = try? kcw.getGenericPasswordFor(
    account: "RWQuickNote",
    service: "unlockPassword") {
    return password
  }

  return ""
}

```

This method creates a `KeychainWrapper` and calls `getGenericPasswordFor(account:service:)` to read the password and return it. The `try?` expression converts an exception to a nil. This will cause the method to return an empty string if the search was unsuccessful.

Next, replace `updateStoredPassword(_:)` with this:

```swift
func updateStoredPassword(_ password: String) {
  let kcw = KeychainWrapper()
  do {
    try kcw.storeGenericPasswordFor(
      account: "RWQuickNote",
      service: "unlockPassword",
      password: password)
  } catch let error as KeychainWrapperError {
    print("Exception setting password: \(error.message ?? "no message")")
  } catch {
    print("An error occurred setting the password.")
  }
}

```

You use `KeychainWrapper` to set a password using the same `account` and `service`. For this app, you’ll print any errors to the console.

Now build and run. The app again asks you to set a password when the app runs. But your app no longer reads from *UserDefaults*, which is not secure. Instead, it uses the encrypted keychain to store and retrieve the password.

![How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/enter-password.png](How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/enter-password.png)

Enter a new password, and you’ll see the note from your earlier run. Tap the lock button twice. This locks and then unlocks the note, using the password you just set. Your password works, and the note appears!

You can now add a password to your keychain, and you can retrieve it to authenticate the user. But you still need to add a few more methods to complete the implementation of Keychain Services in your app.

### Updating a Password in the Keychain

With the app running and the note unlocked, tap the double-arrow button to change the password. Enter a new password and tap *Set Password*. An error appears in the console window:

![How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/add-exception.png](How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/add-exception.png)

You can’t add something to the keychain if the item already exists! Instead, you need to *update* the stored item.

Open *KeychainServices.swift*. Add the following code at the end of your `KeychainWrapper` class:

```swift
func updateGenericPasswordFor(
  account: String,
  service: String,
  password: String
) throws {
  guard let passwordData = password.data(using: .utf8) else {
    print("Error converting value to data.")
    return
  }
  // 1
  let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: account,
    kSecAttrService as String: service
  ]

  // 2
  let attributes: [String: Any] = [
    kSecValueData as String: passwordData
  ]

  // 3
  let status = SecItemUpdate(query as CFDictionary, attributes as CFDictionary)
  guard status != errSecItemNotFound else {
    throw KeychainWrapperError(
      message: "Matching Item Not Found",
      type: .itemNotFound)
  }
  guard status == errSecSuccess else {
    throw KeychainWrapperError(status: status, type: .servicesError)
  }
}

```

This code is similar to `storeGenericPasswordFor(account:service:password:)`, which you added earlier. An update, however, requires two dictionaries, one with the search query and one with the desired changes. Here’s the breakdown:

1. The search query specifies the data you want to update. You provide the attributes as you did with the search query you created earlier, but you don’t use search parameters such as match limit and return attributes. The function will update all matching entries.
2. The second dictionary contains the data to update. You may specify any or all attributes valid for the class, but you only include the ones you want to change. Here you specify only the new password. But you could also set `service` or `account` if you wanted to store new values for those attributes.
3. `SecItemUpdate(_:_:)` uses the contents of the two dictionaries above and performs the update. The most common error you’ll see is *The specified attribute does not exist*. This error indicates that Keychain Services found nothing matching the search query.

You don’t want to have to keep checking and deciding if you need to write a new keychain item or update an existing one, or dealing with the errors that come up if you call the wrong method. You’re going to fix that now.

In `storeGenericPasswordFor(account:service:password:)`, find the `switch status` statement and add a new case above the default:

```
case errSecDuplicateItem:
  try updateGenericPasswordFor(
    account: account,
    service: service,
    password: password)

```

`errSecDuplicateItem` is the status returned when storing an existing item. Now, if you attempt to store an existing item, you’ll fall back to updating it instead.

Build and run. Use the password you set earlier to unlock the note. (Remember that your password change, above, did not complete. The attempt to add an item that already existed caused an exception.) Try again to change the password — success!

Now your app can add, retrieve, and update a password. But there’s still one action missing. You need to be able to *delete* a value from the keychain.

### Deleting a Password From the Keychain

Still in *KeychainServices.swift*, add the following code to the end of `KeychainWrapper`:

```swift
func deleteGenericPasswordFor(account: String, service: String) throws {
  // 1
  let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: account,
    kSecAttrService as String: service
  ]

  // 2
  let status = SecItemDelete(query as CFDictionary)
  guard status == errSecSuccess || status == errSecItemNotFound else {
    throw KeychainWrapperError(status: status, type: .servicesError)
  }
}

```

Again, most of this code is similar to the code you added earlier to add and update an item. Note the changes:

1. Although the query looks much like that for the add and update, here you don’t provide a new value. *Note*: Make sure the query identifies only the item or items you want to delete. If multiple items match the query, Keychain Services deletes *all* of them.
2. You call `SecItemDelete(_:)` using the query to delete the items. There is no undo!

Add the following code to the top of `storeGenericPasswordFor(account:service:password:)` to call your new method:

```
if password.isEmpty {
  try deleteGenericPasswordFor(
    account: account,
    service: service)
  return
}

```

You can now add, retrieve, update, and delete items in a keychain. Those four methods give you the core functionality you need to work with items in your keychain.

Build and run the app and tap the reset button. The app sets the password and note to empty strings. Thanks to the changes you just made, the reset button now deletes the password as well as the note!

Rerun the app and set an initial password. Tap the lock button twice to confirm that your new password unlocks the note. Now tap the arrows button to change the password. Again, tap the lock button twice to verify that the app stored your new password.

![How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/changing-password.png](How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/changing-password.png)

Congratulations! You’ve now added Keychain Services to your app to secure the password.

Typing a password every time to view the note is tedious for the user. Next, you’ll add *biometric authentication* to allow simpler, but still secure, unlocking.

## Biometric Authentication in SwiftUI

Biometric authentication uses unique characteristics of your body to verify your identity. Apple provides two biometric authentication methods:

- *Touch ID* uses your fingerprint.
- *Face ID* uses your face’s unique shape.

Both methods are faster and easier than a password. And they’re more secure. Someone might be able to guess your password, but replicating your fingerprint or face belongs in the world of spy movies rather than in reality!

In the next section, you’ll build biometric authentication into your app.

### Enabling Biometric Authentication in Your App

Open *ToolbarView.swift*. This view defines the toolbar shown above the editor. It includes the lock button, which locks and unlocks the note. This is the perfect place to add biometric authentication to your app. You’ll use the *Local Authentication* framework to do this.

At the top of the file, add the following code after the existing import statement:

```
import LocalAuthentication

```

The `LocalAuthentication` framework allows your app access to the same systems the user has to unlock their device — namely, a passcode, Touch ID or Face ID. Currently, when the note is locked and the user taps the lock button, the app sets the `showUnlockModal` state property to true. Setting `showUnlockModal` to true displays the view that requests and validates the password.

You’ll change this to perform biometric authentication instead. If that fails, or isn’t available, you’ll fall back to the password view.

In *ToolbarView.swift*, add the following method above `body`:

```
func tryBiometricAuthentication() {
  // 1
  let context = LAContext()
  var error: NSError?

  // 2
  if context.canEvaluatePolicy(
    .deviceOwnerAuthenticationWithBiometrics,
    error: &error) {
    // 3
    let reason = "Authenticate to unlock your note."
    context.evaluatePolicy(
      .deviceOwnerAuthenticationWithBiometrics,
      localizedReason: reason) { authenticated, error in
      // 4
      DispatchQueue.main.async {
        if authenticated {
          // 5
          self.noteLocked = false
        } else {
          // 6
          if let errorString = error?.localizedDescription {
            print("Error in biometric policy evaluation: \(errorString)")
          }
          self.showUnlockModal = true
        }
      }
    }
  } else {
    // 7
    if let errorString = error?.localizedDescription {
      print("Error in biometric policy evaluation: \(errorString)")
    }
    showUnlockModal = true
  }
}

```

Here’s how each step works:

1. You access biometric authentication using an `LAContext` object. This gathers the biometrics from the user interaction and communicates with the *Secure Enclave* on the device. `LocalAuthentication` pre-dates Swift and uses Objective-C patterns like *NSError*.
2. You first check that authentication is available. The first parameter, `.deviceOwnerAuthenticationWithBiometrics`, requests biometric authentication.
3. You describe why you want to use authentication in `reason`, which iOS displays to the user. Calling `evaluatePolicy(_:localizedReason:reply:)` requests authentication. The device will perform either Face ID or Touch ID authentication, whichever is available on the current device. The call executes the block when it returns.
4. Since you execute this code from a block and change the UI, you must ensure that the change runs on the UI thread.
5. If authentication succeeded, you set the note as unlocked. Note that you get no further information about the authentication! You find out only that it succeeded or failed.
6. If authentication failed, you print any error that came into the block. You also set the `showUnlockModal` state to true. This action forces your app to fall back to manual password behavior.
7. If the initial check failed, that means biometric authentication is not available. You print the error received and then show the unlock view. Again, you provide a fallback authentication path. 
    
    Some devices do not have authentication, or a user may not have set it up. And authentication can sometimes fail. Whatever the failure reason, you must always provide the user with an appropriate way to get in without biometric authentication!
    

You need one more change to enable this functionality. Press *Command-F* to find the `// Biometric Authentication Point` comment. Replace the next line, `self.showUnlockModal = true`, with a call to the new method:

```
self.tryBiometricAuthentication()

```

Now you’re ready to test your biometric authentication functionality!

### Simulating Biometric Authentication in Xcode

Fortunately, the simulator is able to simulate biometric authentication for you during testing. Your choice of simulated device determines the type of biometric authentication.

### Simulating Touch ID Authentication

Select *iPhone SE (2nd generation)* to test Touch ID. Build and run. Tap the unlock button, and the normal unlock window will appear. If you look in the console, you’ll see an error message: *No identities are enrolled*.

The simulator doesn’t enroll the device in biometric authentication automatically. You have to set it up. In the simulator, select *Features ▸ Touch ID ▸ Enrolled*. Enter your password to unlock the note. Then tap the lock button twice. This will lock it and attempt to unlock it again. This time, you’ll see the *Touch ID* prompt.

![How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/Simulator-Screen-Shot-iPhone-SE-2nd-generation-2020-08-12-at-13.59.49.png](How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/Simulator-Screen-Shot-iPhone-SE-2nd-generation-2020-08-12-at-13.59.49.png)

To simulate biometric authentication, you use the other options under the *Features ▸ Touch ID* menu. To execute a successful authentication, select *Features ▸ Touch ID ▸ Matching Touch*. After a moment, the authentication request will vanish, and the note will unlock.

You can also verify the behavior of a failed authentication. Tap the lock button to lock the note. Tap it again to attempt authentication. To simulate a failed authentication, select *Features ▸ Touch ID ▸ Non-matching Touch*. You’ll see the failed prompt appear:

![How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/Simulator-Screen-Shot-iPhone-SE-2nd-generation-2020-08-12-at-14.00.02.png](How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/Simulator-Screen-Shot-iPhone-SE-2nd-generation-2020-08-12-at-14.00.02.png)

Selecting either option on the prompt will fail the authentication and present the previous unlock page. Enter the password to verify that you can still unlock the note.

You can also let the user enter the device PIN as fallback. To do this, open *ToolbarView.swift* again. Use *Command-F* to find `context.evaluatePolicy`. Change `.deviceOwnerAuthenticationWithBiometrics` to `.deviceOwnerAuthentication`.

Now build and run. Fail the authentication, and notice that the *Enter Password* option has changed to *Enter Passcode*. Tap it, and you’ll see the simulator passcode entry. For the simulator, any passcode value will work. Enter a passcode, and the note will unlock.

![How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/Simulator-Screen-Shot-iPhone-SE-2nd-generation-2020-08-12-at-14.02.22.png](How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/Simulator-Screen-Shot-iPhone-SE-2nd-generation-2020-08-12-at-14.02.22.png)

### Simulating Face ID Authentication

Face ID works much as Touch ID. But Face Id has one new requirement. You must add the *NSFaceIDUsageDescription* key to your app’s *Info.plist* or authorization requests will fail.

Open *Info.plist*. Right-click on an empty part of the page and select *Add Row*. Scroll through the list of keys and select *Privacy – Face ID Usage Description*. Double-click under the value column and enter *To allow you to unlock your note without entering your password.*

Now change the device to the *iPhone 11 Pro*, which supports Face ID. Build and run. In the simulator, select *Features ▸ Face ID ▸ Enrolled*. The menu name has changed to reflect the new biometric authentication method.

In the app on the simulator, choose a new password. Once you’re in the note view, tap the lock button twice. iOS prompts you to allow biometric authentication:

![How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/app-permission.png](How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/app-permission.png)

Tap *OK*. Now you’ll see the *Face ID* prompt. Select *Features ▸ Face ID ▸ Matching Face*, and you’ll see the note unlock.

![How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/face-id-prompt.png](How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/face-id-prompt.png)

### Making the Authentication Method Visible to the User

Now you’ll add one more touch to let the user know the app supports biometric authentication.

In *ToolbarView.swift*, add the following code directly after the `import` statements:

```
func getBiometricType() -> String {
  let context = LAContext()

  _ = context.canEvaluatePolicy(
    .deviceOwnerAuthenticationWithBiometrics,
    error: nil)
  switch context.biometryType {
  case .faceID:
    return "faceid"
  case .touchID:
    // In iOS 14 and later, you can use "touchid" here
    return "lock"
  case .none:
    return "lock"
  @unknown default:
    return "lock"
  }
}

```

This code uses the `biometryType` property on `context` to determine the type of biometric authentication available. It returns the name of an SF Symbol to match or the current lock symbol if one isn’t known. Prior to iOS 14, there isn’t a Touch ID symbol.

Find the `// Lock Icon` comment and change the `Image` code to:

```
Image(systemName: noteLocked ? getBiometricType() : "lock.open")

```

Build and run. Now the lock button indicates that it uses Face ID for authentication!

![How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/faceid-lock-icon.png](How%20To%20Secure%20iOS%20User%20Data%20Keychain%20Services%20and%20%20ef70c7c4dabd4de799486ccb2bbfdb68/faceid-lock-icon.png)

## Where to go from here?

In this tutorial, you’ve learned how to use Keychain Services and how to implement biometric authentication in a SwiftUI app. From here on, your apps will be super secure!

If you wish, you can examine the final project in the project materials. You can download it by clicking the *Download Materials* button at the top or bottom of this tutorial.

You might have protected the password, but the note itself is still stored in plain text in User Defaults. Check out our [CryptoKit tutorial](https://www.raywenderlich.com/10846296-introducing-cryptokit) for information on securing larger amounts of data.

For other classes of keychain items, Apple’s documentation for [Keychain Services](https://developer.apple.com/documentation/security/keychain_services/keychain_items) should be your first stop. The process for other classes works much the same, and the documentation contains Swift examples of most of them.

For similar material from a UIKit perspective, read our article [How To Secure iOS User Data: The Keychain and Biometrics — Face ID or Touch ID](https://www.raywenderlich.com/236-how-to-secure-ios-user-data-the-keychain-and-biometrics-face-id-or-touch-id).

And for another perspective on Keychain, look at [Keychain Services API Tutorial for Passwords in Swift](https://www.raywenderlich.com/9240-keychain-services-api-tutorial-for-passwords-in-swift).