# Keychain Services API Tutorial for Passwords in Swift | Kodeco

## Getting Started

For this tutorial, you’ll use ***SecureStore***, a boilerplate iOS framework where you’ll implement your ***Keychain Services API***.

Start by downloading the starter project using the ***Download Materials*** button at the top or bottom of this tutorial. Once you’ve downloaded it, open ***SecureStore.xcodeproj*** in Xcode.

To keep you focused, the starter project has everything related to implementing your wrapper already set up for you.

The structure of your project should look like this:

![Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/keychain_project_structure-1.png](Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/keychain_project_structure-1.png)

The code of your wrapper lives in the ***SecureStore*** group folder:

- ***SecureStoreError.swift***: Contains an `enum`, which represents all the possible errors your wrapper can deal with. Conforming to `LocalizedError`, `SecureStoreError` provides localized messages describing the error and why it occurred.
- ***SecureStoreQueryable.swift***: Defines a protocol with the same name as the file. `SecureStoreQueryable` forces the implementer to provide a `query` property defined as a dictionary typed as `[String: Any]`. Internally, your API only deals with those types of objects. More on that later.
- ***SecureStore.swift***: Defines the wrapper you’ll implement in this tutorial. It provides an initializer and a bunch of stubbed methods for adding, updating, deleting and retrieving your passwords from the ***Keychain***. A consumer can create a wrapper’s instance by injecting some type that conforms to `SecureStoreQueryable`.
- ***InternetProtocol.swift***: Represents all the possible internet protocol values you can deal with.
- ***InternetAuthenticationType.swift***: Describes the authentication mechanisms that your wrapper provides.

<aside>
💡 *Note*: *Dependency injection* allows you to write classes that expand and isolate functionality. It’s a bit of a scary word for a pretty simple concept. You’ll see the word “inject” throughout this tutorial, where it refers to passing a whole object into an initializer.

</aside>

Along with the framework code, you should have two other folders: *SecureStoreTests* and *TestHost*. The former contains the unit tests you’ll ship with your framework. The latter contains an empty app, which you’ll use to test your framework API.

*Note*: Usually, to test the code you write in a tutorial, you run an app in the simulator. Instead of doing that, you’ll verify your code is working by running unit tests. So the test host app in the project won’t run in the simulator; instead, it serves as the container where it executes unit tests for your framework.

Before diving directly into the code, take a look at some theory!

## An Overview of Keychain Services

Why use the *Keychain* over simpler solutions? Wouldn’t storing the user’s base-64 encoded password in `UserDefaults` be enough?

Definitely not! It’s trivial for an attacker to recover a password stored that way.

*Keychain Services* help you to securely store items, or small chunks of data, into an encrypted database on behalf of the user.

From Apple’s documentation, the `[SecKeychain](https://developer.apple.com/documentation/security/seckeychain)` class represents a database, while the `[SecKeychainItem](https://developer.apple.com/documentation/security/seckeychainitem)` class represents an item.

*Keychain Services* operate differently depending on the operating system you’re running.

In iOS, apps have access to a single ***Keychain*** which includes the *iCloud Keychain*. Locking and unlocking the device automatically locks and unlocks ***Keychain***. This prevents unwanted accesses. Furthermore, an app is only able to access its own items or those shared with a group to which it belongs.

On the other hand, macOS supports multiple keychains. You typically rely on the user to manage these with the *Keychain Access* app and work implicitly with the default keychain. Additionally, you can manipulate keychains directly; for example, creating and managing a keychain that is strictly private to your app.

When you want to store a secret such as a password, you package it as a keychain item. This is an opaque type which consists of two parts: data and a set of attributes. Just before it inserts a new item, *Keychain Services* encrypts the data then wraps it together with its attributes.

![Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/keychain_services_encryption-1.png](Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/keychain_services_encryption-1.png)

Use attributes to identify and store metadata or to control access to your stored items. Specify attributes as the keys and values of a dictionary expressed as a `CFDictionary`. You can find a list of the available keys at [Item Attribute Keys and Values](https://developer.apple.com/documentation/security/keychain_services/keychain_items/item_attribute_keys_and_values). The corresponding values can be strings, numbers, some other basic types, or constants packaged with the [Security](https://developer.apple.com/documentation/security) framework.

*Keychain Services* provide special kinds of attributes that allow you to identify the class for a specific item. In this tutorial, you’ll use both `[kSecClassGenericPassword](https://developer.apple.com/documentation/security/ksecclassgenericpassword)` and `[kSecClassInternetPassword](https://developer.apple.com/documentation/security/ksecclassinternetpassword)` to deal with generic and internet passwords.

Each class supports only a special set of attributes. In other words, not all attributes apply to a specific item class. You can verify them in the relevant [item class value documentation](https://developer.apple.com/documentation/security/keychain_services/keychain_items/item_class_keys_and_values).

*Note*: Besides manipulating passwords, Apple offers the chance to interact with other types of items like certificates, cryptographic keys and identities. Those are represented respectively by the `[kSecClassCertificate](https://developer.apple.com/documentation/security/ksecclasscertificate)`, `[kSecClassKey](https://developer.apple.com/documentation/security/ksecclasskey)` and `[kSecClassIdentity](https://developer.apple.com/documentation/security/ksecclassidentity)` classes.

## Diving Into Keychain Services API

Since the code hides items from ill-intentioned users, *Keychain Services* provide a set of C functions to interact with. Here are the APIs you’ll use to manipulate both generic and internet passwords:

- *[SecItemAdd(_:_:)](https://developer.apple.com/documentation/security/1401659-secitemadd)*: Use this function to add one or more items to a keychain.
- *[SecItemCopyMatching(_:_:)](https://developer.apple.com/documentation/security/1398306-secitemcopymatching)*: This function returns one or more keychain items that match a search query. Additionally, it can copy attributes of specific keychain items.
- *[SecItemUpdate(_:_:)](https://developer.apple.com/documentation/security/1393617-secitemupdate)*: This function allows you to modify items that match a search query.
- *[SecItemDelete(_:)](https://developer.apple.com/documentation/security/1395547-secitemdelete)*: This function removes items that match a search query.

While the functions above operate with different parameters, they all return a result code expressed as an `OSStatus`. This is a 32-bit signed integer which can assume one of the values listed in [Item Return Result Keys](https://developer.apple.com/documentation/security/1542001-security_framework_result_codes).

Since `OSStatus` could be cryptic to understand, Apple provides an additional API called `SecCopyErrorMessageString(_:_:)` to obtain a human-readable string corresponding to these status codes.

*Note*: Aside from adding, modifying, deleting or searching for a specific keychain item, Apple also provides functions to both export and import certificates, keys and identities or even modify items’ access control. If you want to learn more, check out the documentation for [Keychain Items](https://developer.apple.com/documentation/security/keychain_services/keychain_items).

Now that you have a solid grasp of *Keychain Services*, in the next section you’ll learn how to remove the stubbed methods provided by your wrapper.

## Implementing Wrapper’s API

Open *SecureStore.swift* and add the following implementation inside `setValue(_:for:)`:

```swift
// 1
guard let encodedPassword = value.data(using: .utf8) else {
  throw SecureStoreError.string2DataConversionError
}

// 2
var query = secureStoreQueryable.query
query[String(kSecAttrAccount)] = userAccount

// 3
var status = SecItemCopyMatching(query as CFDictionary, nil)
switch status {
// 4
case errSecSuccess:
  var attributesToUpdate: [String: Any] = [:]
  attributesToUpdate[String(kSecValueData)] = encodedPassword

  status = SecItemUpdate(query as CFDictionary,
                         attributesToUpdate as CFDictionary)
  if status != errSecSuccess {
    throw error(from: status)
  }
// 5
case errSecItemNotFound:
  query[String(kSecValueData)] = encodedPassword

  status = SecItemAdd(query as CFDictionary, nil)
  if status != errSecSuccess {
    throw error(from: status)
  }
default:
  throw error(from: status)
}

```

This method, as the name implies, allows storing a new password for a specific account. If it cannot update or add a password, it throws a `SecureStoreError.unhandledError`, which specifies a localized description for it.

Here’s what your code does:

1. Check if it can encode the value to store into a `Data` type. If that’s not possible, it throws a conversion error.
2. Ask the `secureStoreQueryable` instance for the query to execute and append the account you’re looking for.
3. Return the keychain item that matches the query.
4. If the query succeeds, it means a password for that account already exists. In this case, you replace the existing password’s value using `SecItemUpdate(_:_:)`.
5. If it cannot find an item, the password for that account does not exist yet. You add the item by invoking `SecItemAdd(_:_:)`.

The *Keychain Services API* uses Core Foundation types. To make the compiler happy, you must convert from Core Foundation types to Swift types and vice versa.

In the first case, since each key’s attribute is of type `CFString`, its usage as a key in a query dictionary requires a cast to `String`. However, the conversion from `[String: Any]` to `CFDictionary` enables you to invoke the C functions.

Now it’s time to retrieve your password. Scroll below the method you’ve just implemented and replace the implementation of `getValue(for:)` with the following:

```swift
// 1
var query = secureStoreQueryable.query
query[String(kSecMatchLimit)] = kSecMatchLimitOne
query[String(kSecReturnAttributes)] = kCFBooleanTrue
query[String(kSecReturnData)] = kCFBooleanTrue
query[String(kSecAttrAccount)] = userAccount

// 2
var queryResult: AnyObject?
let status = withUnsafeMutablePointer(to: &queryResult) {
  SecItemCopyMatching(query as CFDictionary, $0)
}

switch status {
// 3
case errSecSuccess:
  guard
    let queriedItem = queryResult as? [String: Any],
    let passwordData = queriedItem[String(kSecValueData)] as? Data,
    let password = String(data: passwordData, encoding: .utf8)
    else {
      throw SecureStoreError.data2StringConversionError
  }
  return password
// 4
case errSecItemNotFound:
  return nil
default:
  throw error(from: status)
}

```

Given a specific account, this method retrieves the password associated with it. Again, if something goes wrong with the request, the code throws a `SecureStoreError.unhandledError`.

Here’s what’s happening with the code you’ve just added:

1. Ask `secureStoreQueryable` for the query to execute. Besides adding the account you’re interested in, this enriches the query with other attributes and their related values. In particular, you’re asking it to return a single result, to return all the attributes associated with that specific item and to give you back the unencrypted data as a result.
2. Use `SecItemCopyMatching(_:_:)` to perform the search. On completion, `queryResult` will contain a reference to the found item, if available. `withUnsafeMutablePointer(to:_:)` gives you access to an `UnsafeMutablePointer` that you can use and modify inside the closure to store the result.
3. If the query succeeds, it means that it found an item. Since the result is represented by a dictionary that contains all the attributes you’ve asked for, you need to extract the data first and then decode it into a `Data` type.
4. If an item is not found, return a `nil` value.

Adding or retrieving passwords for an account is not enough. You need to integrate a way to remove passwords as well.

Find `removeValue(for:)` and add this implementation:

```swift
var query = secureStoreQueryable.query
query[String(kSecAttrAccount)] = userAccount

let status = SecItemDelete(query as CFDictionary)
guard status == errSecSuccess || status == errSecItemNotFound else {
  throw error(from: status)
}

```

To remove a password, you perform `SecItemDelete(_:)` specifying the account you’re looking for. If you successfully deleted the password or if no item was found, your job is done and you bail out. Otherwise, you throw an unhandled error in order to let the user know something went wrong.

But what if you want to remove all the passwords associated with a specific service? Your next step is to implement the final code for achieving this.

Find `removeAllValues()` and add the following code within its brackets:

```
let query = secureStoreQueryable.query

let status = SecItemDelete(query as CFDictionary)
guard status == errSecSuccess || status == errSecItemNotFound else {
  throw error(from: status)
}

```

As you’ll notice, this method is similar to the previous one except for the query passed to the `SecItemDelete(_:)` function. In this case, you remove passwords independently from the user account.

Finally, build the framework to verify everything compiles correctly.

## Connecting the Dots

All the work you’ve done so far enriches your wrapper with add, update, delete and retrieve capabilities. As is, you must create the wrapper with an instance of some type that conforms to `SecureStoreQueryable`.

Since your very first goal was to deal both with generic and internet passwords, your next step is to create two different configurations that a consumer can create and inject into your wrapper.

First, examine how to compose a query for generic passwords.

Open *SecureStoreQueryable.swift* and add the following code below the `SecureStoreQueryable` definition:

```
public struct GenericPasswordQueryable {
  let service: String
  let accessGroup: String?

  init(service: String, accessGroup: String? = nil) {
    self.service = service
    self.accessGroup = accessGroup
  }
}

```

`GenericPasswordQueryable` is a simple `struct` that accepts a service and an access group as `String` parameters.

Next, add the following extension below the `GenericPasswordQueryable` definition:

```
extension GenericPasswordQueryable: SecureStoreQueryable {
  public var query: [String: Any] {
    var query: [String: Any] = [:]
    query[String(kSecClass)] = kSecClassGenericPassword
    query[String(kSecAttrService)] = service
    // Access group if target environment is not simulator
    #if !targetEnvironment(simulator)
    if let accessGroup = accessGroup {
      query[String(kSecAttrAccessGroup)] = accessGroup
    }
    #endif
    return query
  }
}

```

To conform to `SecureStoreQueryable` protocol, you must implement `query` as a property. The query represents the way your wrapper is able to perform the chosen functionality.

The composed query has specific keys and values:

- The item class, represented by the key `kSecClass`, has the value `kSecClassGenericPassword` since you’re dealing with generic passwords. This is how keychain infers that the data is secret and requires encryption.
- `kSecAttrService` is set to the `service` parameter value that is injected with a new instance of `GenericPasswordQueryable`.
- Finally, if your code is not running on a simulator, you also set `kSecAttrAccessGroup` key to the provided `accessGroup` value. This lets you share items among different apps with the same access group.

Next, build the framework to ensure that everything works correctly.

*Note*: For a keychain item of class `kSecClassGenericPassword`, the primary key is the combination of `kSecAttrAccount` and `kSecAttrService`. In other words, the tuple allows you to uniquely identify a generic password in the *Keychain*.

Your shiny new wrapper is not complete yet! The next step is to integrate the functionalities allowing consumers to interact with internet passwords as well.

Scroll to the end of *SecureStoreQueryable.swift* and add the following:

```
public struct InternetPasswordQueryable {
  let server: String
  let port: Int
  let path: String
  let securityDomain: String
  let internetProtocol: InternetProtocol
  let internetAuthenticationType: InternetAuthenticationType
}

```

`InternetPasswordQueryable` is a `struct` that helps you manipulate *Internet Passwords* within your applications *Keychain*.

Before conforming to `SecureStoreQueryable`, take a moment to understand how your API will work in this case.

If users want to deal with internet passwords, they create a new instance of `InternetPasswordQueryable` where `internetProtocol` and `internetAuthenticationType` properties are bound to specific domains.

Next, add the following to below your `InternetPasswordQueryable` implementation:

```
extension InternetPasswordQueryable: SecureStoreQueryable {
  public var query: [String: Any] {
    var query: [String: Any] = [:]
    query[String(kSecClass)] = kSecClassInternetPassword
    query[String(kSecAttrPort)] = port
    query[String(kSecAttrServer)] = server
    query[String(kSecAttrSecurityDomain)] = securityDomain
    query[String(kSecAttrPath)] = path
    query[String(kSecAttrProtocol)] = internetProtocol.rawValue
    query[String(kSecAttrAuthenticationType)] = internetAuthenticationType.rawValue
    return query
  }
}

```

As seen in the generic passwords case, the query has specific keys and values:

- The item class, represented by the key `kSecClass`, has the value `kSecClassInternetPassword`, since you’re now interacting with internet passwords.
- `kSecAttrPort` is set to the `port` parameter.
- `kSecAttrServer` is set to the `server` parameter.
- `kSecAttrSecurityDomain` is set to the `securityDomain` parameter.
- `kSecAttrPath` is set to the `path` parameter.
- `kSecAttrProtocol` is bound to the `rawValue` of the `internetProtocol` parameter.
- Finally, `kSecAttrAuthenticationType` is bound to the `rawValue` of the `internetAuthenticationType` parameter.

Again, build to see if Xcode compiles correctly.

*Note*: For a keychain item of class `kSecClassInternetPassword`, the primary key is the combination of `kSecAttrAccount`, `kSecAttrSecurityDomain`, `kSecAttrServer`, `kSecAttrProtocol`, `kSecAttrAuthenticationType`, `kSecAttrPort` and `kSecAttrPath`. In other words, those values allow you to uniquely identify an internet password in the *Keychain*.

Now it’s time to see the result of all your hard work. But wait! Since you’re not creating an app that runs on a simulator, how are you going to verify it?

Here’s where unit tests come to the rescue.

## Testing the Behavior

In this section, you’ll see how to integrate unit tests for your wrapper. In particular, you’ll test the functionalities that your wrapper exposes.

*Note*: If you’re new to unit tests and you want to explore the subject, check out our amazing [iOS Unit Testing and UI Testing Tutorial](https://www.raywenderlich.com/709-ios-unit-testing-and-ui-testing-tutorial).

### Creating the Class

To create the class that will contain all your unit tests, click *File* ▸ *New* ▸ *File…* and select *iOS* ▸ *Source* ▸ *Unit Test Case Class*. On the next screen, specify the class name as *SecureStoreTests*, subclass *XCTestCase* and make sure the language is *Swift*. Click *Next*, choose the *SecureStoreTests* group, verify that you have selected the *SecureStoreTests* targets checkbox and click *Create*.

Xcode will prompt a dialog to create an Objective-C bridging header. Click *Don’t Create* to skip the creation.

![Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/keychain_bridging_header-1-650x329.png](Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/keychain_bridging_header-1-650x329.png)

Open *SecureStoreTests.swift* file and remove all the code within the curly braces.

Next, add the following below the `import XCTest` statement:

```
@testable import SecureStore

```

This gives the unit tests access to the classes and methods defined in your SecureStore framework.

*Note*: You might see a “No such module” error. Don’t worry, the error will go away when you get to the end of this section of the tutorial and perform the test.

Next, add the following properties at the top of `SecureStoreTests`:

```
var secureStoreWithGenericPwd: SecureStore!
var secureStoreWithInternetPwd: SecureStore!

```

Next, add a new `setUp()` method like this:

```
override func setUp() {
  super.setUp()

  let genericPwdQueryable =
    GenericPasswordQueryable(service: "someService")
  secureStoreWithGenericPwd =
    SecureStore(secureStoreQueryable: genericPwdQueryable)

  let internetPwdQueryable =
    InternetPasswordQueryable(server: "someServer",
                              port: 8080,
                              path: "somePath",
                              securityDomain: "someDomain",
                              internetProtocol: .https,
                              internetAuthenticationType: .httpBasic)
  secureStoreWithInternetPwd =
    SecureStore(secureStoreQueryable: internetPwdQueryable)
}

```

Since you test both generic and internet passwords, you create the two instances of your wrapper with two different configurations. Those configurations are the ones you developed in the previous section.

Before you forget, you’ll want to clear the state of the keychain during the tear down phase of the test so that you can start fresh for next time. Add this method to the end of the class:

```
override func tearDown() {
  try? secureStoreWithGenericPwd.removeAllValues()
  try? secureStoreWithInternetPwd.removeAllValues()

  super.tearDown()
}

```

Since you should isolate and execute each test independently from the others, you’re going to delete all the passwords already available in the *Keychain*. The execution order doesn’t matter.

It’s now time to add unit tests for generic passwords.

### Testing Generic Passwords

Add the following code below `tearDown()`:

```
// 1
func testSaveGenericPassword() {
  do {
    try secureStoreWithGenericPwd.setValue("pwd_1234", for: "genericPassword")
  } catch (let e) {
    XCTFail("Saving generic password failed with \(e.localizedDescription).")
  }
}

// 2
func testReadGenericPassword() {
  do {
    try secureStoreWithGenericPwd.setValue("pwd_1234", for: "genericPassword")
    let password = try secureStoreWithGenericPwd.getValue(for: "genericPassword")
    XCTAssertEqual("pwd_1234", password)
  } catch (let e) {
    XCTFail("Reading generic password failed with \(e.localizedDescription).")
  }
}

// 3
func testUpdateGenericPassword() {
  do {
    try secureStoreWithGenericPwd.setValue("pwd_1234", for: "genericPassword")
    try secureStoreWithGenericPwd.setValue("pwd_1235", for: "genericPassword")
    let password = try secureStoreWithGenericPwd.getValue(for: "genericPassword")
    XCTAssertEqual("pwd_1235", password)
  } catch (let e) {
    XCTFail("Updating generic password failed with \(e.localizedDescription).")
  }
}

// 4
func testRemoveGenericPassword() {
  do {
    try secureStoreWithGenericPwd.setValue("pwd_1234", for: "genericPassword")
    try secureStoreWithGenericPwd.removeValue(for: "genericPassword")
    XCTAssertNil(try secureStoreWithGenericPwd.getValue(for: "genericPassword"))
  } catch (let e) {
    XCTFail("Saving generic password failed with \(e.localizedDescription).")
  }
}

// 5
func testRemoveAllGenericPasswords() {
  do {
    try secureStoreWithGenericPwd.setValue("pwd_1234", for: "genericPassword")
    try secureStoreWithGenericPwd.setValue("pwd_1235", for: "genericPassword2")
    try secureStoreWithGenericPwd.removeAllValues()
    XCTAssertNil(try secureStoreWithGenericPwd.getValue(for: "genericPassword"))
    XCTAssertNil(try secureStoreWithGenericPwd.getValue(for: "genericPassword2"))
  } catch (let e) {
    XCTFail("Removing generic passwords failed with \(e.localizedDescription).")
  }
}

```

There’s quite a bit going on here, so breaking it down:

1. `testSaveGenericPassword()` methods verifies whether it can save a password correctly.
2. `testReadGenericPassword()` first saves the password then retrieves the password, checking if it’s equal to the expected one.
3. `testUpdateGenericPassword()` verifies when saving a different password for the same account, the latest password is the one expected after its retrieval.
4. `testRemoveGenericPassword()` tests that it can remove a password for a specific account.
5. Finally, `testRemoveAllGenericPasswords` checks that all the passwords related to a specific service are deleted from the *Keychain*.

Since your wrapper can throw exceptions, each `catch` block makes the tests fail if something goes wrong.

### Checking Your Work

Now it’s time to verify that everything works as expected. Select *TestHost* as the active scheme for your Xcode project:

![Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/testhost-scheme.png](Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/testhost-scheme.png)

Press **Command-U** on your keyboard (or select *Product ▸ Test* in the menu) to perform the unit tests.

*Note*: You don’t need to run the app as you normally would in a tutorial. For this tutorial, you check your code by performing the unit tests.

Show the Test navigator and wait for the tests to execute. Once they’ve finished, you’ll expect all five tests to be green. Nice!

![Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/keychain_green_tests.png](Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/keychain_green_tests.png)

Next, do the same for internet passwords.

Scroll to the end of the class and just before the last curly brace add the following:

```
func testSaveInternetPassword() {
  do {
    try secureStoreWithInternetPwd.setValue("pwd_1234", for: "internetPassword")
  } catch (let e) {
    XCTFail("Saving Internet password failed with \(e.localizedDescription).")
  }
}

func testReadInternetPassword() {
  do {
    try secureStoreWithInternetPwd.setValue("pwd_1234", for: "internetPassword")
    let password = try secureStoreWithInternetPwd.getValue(for: "internetPassword")
    XCTAssertEqual("pwd_1234", password)
  } catch (let e) {
    XCTFail("Reading internet password failed with \(e.localizedDescription).")
  }
}

func testUpdateInternetPassword() {
  do {
    try secureStoreWithInternetPwd.setValue("pwd_1234", for: "internetPassword")
    try secureStoreWithInternetPwd.setValue("pwd_1235", for: "internetPassword")
    let password = try secureStoreWithInternetPwd.getValue(for: "internetPassword")
    XCTAssertEqual("pwd_1235", password)
  } catch (let e) {
    XCTFail("Updating internet password failed with \(e.localizedDescription).")
  }
}

func testRemoveInternetPassword() {
  do {
    try secureStoreWithInternetPwd.setValue("pwd_1234", for: "internetPassword")
    try secureStoreWithInternetPwd.removeValue(for: "internetPassword")
    XCTAssertNil(try secureStoreWithInternetPwd.getValue(for: "internetPassword"))
  } catch (let e) {
    XCTFail("Removing internet password failed with \(e.localizedDescription).")
  }
}

func testRemoveAllInternetPasswords() {
  do {
    try secureStoreWithInternetPwd.setValue("pwd_1234", for: "internetPassword")
    try secureStoreWithInternetPwd.setValue("pwd_1235", for: "internetPassword2")
    try secureStoreWithInternetPwd.removeAllValues()
    XCTAssertNil(try secureStoreWithInternetPwd.getValue(for: "internetPassword"))
    XCTAssertNil(try secureStoreWithInternetPwd.getValue(for: "internetPassword2"))
  } catch (let e) {
    XCTFail("Removing internet passwords failed with \(e.localizedDescription).")
  }
}

```

Notice that the code above is identical to the one previously analyzed. You’ve just replaced the reference `secureStoreWithGenericPwd` with `secureStoreWithInternetPwd`.

Select *TestHost* as the active scheme, if it’s not already, and press *Command-U* on your keyboard to test again. Now all the tests, both for generic and internet passwords, should be green.

Congratulations! You now have a working, stand-alone framework and unit tests in place.

## Where to Go From Here?

In this tutorial, you made a framework wrapping the *Keychain Services API* and even integrated unit tests to prove your code works as expected. Amazing!

You could go a step further, sharing or distributing your code with other people following the final part of our tutorial [Creating a Framework for iOS](https://www.raywenderlich.com/5109-creating-a-framework-for-ios).

You can download the completed version of the project using the *Download Materials* button at the top or bottom of this tutorial.

If you want to learn more, check out Apple’s documentation at [Keychain Services](https://developer.apple.com/documentation/security/keychain_services).

It’s worth noting that *Keychain* is not limited to passwords. You can store sensitive information like credit card data or short notes. You can also save items like cryptographic keys and certificates that you manage with [Certificate, Key, and Trust Services](https://developer.apple.com/documentation/security/certificate_key_and_trust_services), to conduct secure and authenticated data transactions.

What did you learn from this? Any lingering questions? Want to share something that happened along the way? You can discuss it in the forums. See you there!

![Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/d5d6a5418e689f22ebdd7741398ca033.jpg](Keychain%20Services%20API%20Tutorial%20for%20Passwords%20in%20Sw%2081f064e8ebb64304bd66d105b7c781ab/d5d6a5418e689f22ebdd7741398ca033.jpg)