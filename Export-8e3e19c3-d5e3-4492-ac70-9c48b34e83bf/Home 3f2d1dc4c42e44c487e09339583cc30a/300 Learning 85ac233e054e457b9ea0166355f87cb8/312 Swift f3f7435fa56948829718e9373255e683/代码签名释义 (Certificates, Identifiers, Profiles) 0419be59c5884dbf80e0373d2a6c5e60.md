# 代码签名释义 (Certificates, Identifiers, Profiles)

# **What is it and why do we need it?**

Fundamentally, we could define iOS Code Signing as follows:

> Code Signing controls which **apps**, made by which **developer**, can run on which **devices
代码签名控制** 其实就是那个**apps**，是由哪个**开发者**开发， 可以安装在哪些**设备**上
> 

We can see that there are three main aspects:

1. **Apps**📦 – We need a way to uniquely identify an application
2. **Developers**👩‍💻👨‍💻– A developer (or group thereof) must be able to prove their identity
3. **Devices**📱⌚️ – An app can be distributed in different ways, and iOS must be able to identify which ways are allowed and which are not

# ****How do we achieve that?****

Three problems, three components to solve them: Bundle Identifiers, Certificates and Provisioning Profiles.

1. We use a **Bundle Identifier**🏷 which is just a string, typically of the form `<country>.<developer>.<project>` to identify an application. For example the Migros Play app uses `ch.migros.play`. Since the same Bundle Identifier could theoretically be used by different developers, this is automatically prefixed by Code Signing with the **Team Identifier** which is created during the registration of a developer with Apple. The final Bundle ID of an app looks something like `ASDFGH1234.ch.migros.play`
2. A developer can create **Certificates**📜 which enable them to prove their identity to Apple, or to iOS respectively.
3. Now to tie it all together and enable our app to run, we use **Provisioning Profiles**✉️. A provisioning profile specifies a Bundle Identifier, so we know which app the permission is for, a Certificate, so we know who created the app, and it also defines in which ways the app can be distributed.

There are different kinds of certificates and provisioning profiles, and together, they are used for different purposes. Let's explore which types of *signing assets* (certificates and provisioning profiles) you need to achieve that.

# 对应场景 / **Environments 🌍**

## ****Development 🛠 – Simulator 💻****

Let's start with an exception – but in favor of simplicity. To run an app on the iOS Simulator that comes with Xcode, you don't need any code signing at all! Just hit "Run" and the app will run.

## ****Development 🛠 – Real Device📱****

*Certificate: Development*

*Provisioning Profile: Development*

But of course, testing the on a real device is better because it is a more accurate representation of the environment it will be running in once you distribute the app. Signing your app for *Development* allows you to build it in Xcode and directly run it on a device – be it iOS, watchOS or tvOS. The device has to be physically connected to your machine with a cable or over the network, and the app will be installed directly onto it. For this, you need a *Development Certificate*, along with a *Development Provisioning Profile*, both of which are needed to run the app on a device.

## ****Distribution 📤 Ad Hoc 👥****

*Certificate: Distribution*

*Provisioning Profile: Ad Hoc*

Distributing your app means it runs on more and more devices, and more people can be exposed to apps that do malicious things, so here is where things get more restrictive. Let's take a look at Ad Hoc distribution. This kind of signing allows your app to run on a **specific set of devices** that you have to **register in advance**. So if you have a small group of test devices, this is a practical way to test your applications on just those devices. Registering your device is done in the [Apple Developer Portal](https://developer.apple.com/), and it requires you to submit a *unique device identifier (UDID)*.

Check out our article on how to register your devices for Ad Hoc Signing.

# **Fastlane/快车道：无需注册设备安装**

If you want to run your app on many devices, this can get a bit cumbersome, right? We'll show you how you can simplify registering the devices using `Fastlane` somewhere down the road. But what if we don't want to deal with any of this, and let our app run on many devices without registering them in advance? This leads us to...

## ****Distribution 📤 – Enterprise🏢****

*Certificate: Distribution (Enterprise)*

*Provisoning Profile: Universal Distribution*

For some of our projects, we want to have large groups of testers and of course asking every one of them to provide their UDID is highly impractical. (Besides Apple not supporting more than 100 Ad Hoc devices for an app.) So we can circumvent this by using an *Enterprise* account. This is a separate account you have to make, which costs a bit more than a standard developer account. It does not have the possibility to deploy apps on the regular App Store. But get this: With Enterprise signing, you can run your apps on any device, without having to register the UDIDs in advance!

To achieve this, you need to use the enterprise account to generate a specific set of certificates and provisioning profiles. Generally, if you have lots of apps that you want to distribute, it's best to make a wildcard provisioning profile that allows any app to be signed with it.

But be aware you can only use a Enterprise profile within your Organization.

## ****Distribution📤 – App Store🌎****

*Certificate: Distribution*

*Provisioning Profile: Distribution*

To distribute your app in the App Store, it needs to pass the Apple App Review. This way, we also don't have to register the devices that can run the app beforehand – similar to Enterprise signing. Apps that are on the store are implicitly trusted. You just sign it with your distribution certificate and provisioning profile, upload it to App Store Connect and you're good to go – for as long as your Developer Membership at Apple is valid.

# **How to Code Sign for Updraft**

When we use Updraft internally, we often choose to use our Enterprise certificates because it provides the most hassle-free experience once everything is set up. If you are using a regular developer account, you probably prefer to use Ad Hoc signing since it comes with your developer account without any additional costs. Make sure to pre-register all the devices that you want to run the app on. To get your UDID just click on [this link](https://getupdraft.com/get-my-udid).

# **Summary**

In this article we have explained what Code Signing is all about and why it's needed to run your apps. We've also explored the different components and how they come together to distribute your apps in different ways. If you are having trouble with Code Signing or getting your apps to run when distributed with Updraft, feel free to [contact us](http://docs.getupdraft.com/contact) or read all further information in our [blog post](https://getupdraft.com/blog/ios-code-signing-development-and-distribution-prov) and we'll do our best to get you up and runnin