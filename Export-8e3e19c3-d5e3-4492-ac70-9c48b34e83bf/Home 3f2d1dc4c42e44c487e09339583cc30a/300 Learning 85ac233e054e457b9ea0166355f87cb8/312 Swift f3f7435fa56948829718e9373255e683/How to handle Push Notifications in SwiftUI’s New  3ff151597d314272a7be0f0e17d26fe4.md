# How to handle Push Notifications in SwiftUI’s New App Lifecycle?

## Xcode 12 — iOS 14

With Xcode 12, Apple have introduced new SwiftUI app life cycle. Which looks something like following:

```swift
@main
struct ProjectName: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

In this tutorial you will learn how to manage push with above app life cycle:

- Ask for Push permission
- Get Push Token
- Handle Notification taps

## **Ask for Push permission**

This process is fairly same as older iOS. All you need to do is request authorisation from *UNUserNotificationCenter* like following:

```swift
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound]) { (allowed, error)in
//This callback does not trigger on main loop be careful
if allowed {
      os_log(.debug, "Allowed") //import os
    }else {
      os_log(.debug, "Error")
    }
}
```

This can be called from any Content view or Observable object.

## **Get Push Token**

*didRegisterForRemoteNotificationsWithDeviceToken* call-back gives Push token but it was part of Appdelegate. So we need to get access to appdelegate. To get Appdelegate instance, you need to add *UIApplicationDelegateAdaptor* property to app entry point:

```swift
@main
struct ios14_demoApp: App {
   @UIApplicationDelegateAdaptorprivatevar appDelegate: AppDelegate
var body:some Scene {
     WindowGroup {
        Text("Demo")
     }
   }
}//*** Implement App delegate ***//class AppDelegate: NSObject, UIApplicationDelegate {
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey :Any]? =nil) -> Bool {
returntrue
}//No callback in simulator
//-- must use device to get valid push token
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data)          {
      print(deviceToken)
    }     func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {
       print(error.localizedDescription)
    }}
```

## **Handle Notification taps**

Now that you have push token on user permission. Assuming, by following [this](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1)apple documentation you are able to send push back to device. the last step is to handle this push action.

All the callback for Push action come in *UNUserNotificationCenterDelegate.* All you need to do is to create a *StateObject* which can handle delegate action like following:

```swift
class NotificationCenter: NSObject, ObservableObject {
var dumbData: UNNotificationResponse?
overrideinit() {
super.init()
       UNUserNotificationCenter.current().delegate =self
}
}extension NotificationCenter: UNUserNotificationCenterDelegate  {func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler:@escaping (UNNotificationPresentationOptions) -> Void) {
   completionHandler([.alert, .sound, .badge])
}func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler:@escaping () -> Void) {
   dumbData = response
   completionHandler()}  func userNotificationCenter(_ center: UNUserNotificationCenter, openSettingsFor notification: UNNotification?) { }}
```

You can go through this [apple document](https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate) to know which delegate method called when to handle notification accordingly.

## **Here is a sample gist which handles local notification and permissions:**

```swift
import SwiftUI
import os
@main
struct ios14DemoApp: App {
    @StateObject var notificationCenter = NotificationCenter()

    @UIApplicationDelegateAdaptor private var appDelegate: AppDelegate

    var body: some Scene {
        WindowGroup {
            LocalNotificationDemoView(notificationCenter: notificationCenter)
        }
    }
}

class AppDelegate: NSObject, UIApplicationDelegate {
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
        return true
    }
    //No callback in simulator -- must use device to get valid push token
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        print(deviceToken)
    }
    
    func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {
        print(error.localizedDescription)
    }
}

class NotificationCenter: NSObject, ObservableObject {
    @Published var dumbData: UNNotificationResponse?
    
    override init() {
        super.init()
        UNUserNotificationCenter.current().delegate = self
    }
}

extension NotificationCenter: UNUserNotificationCenterDelegate  {
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.alert, .sound, .badge])
    }

    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        dumbData = response
        completionHandler()
    }

    func userNotificationCenter(_ center: UNUserNotificationCenter, openSettingsFor notification: UNNotification?) { }
}

class LocalNotification: ObservableObject {
    init() {
        
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound]) { (allowed, error) in
            //This callback does not trigger on main loop be careful
            if allowed {
                os_log(.debug, "Allowed")
            } else {
                os_log(.debug, "Error")
            }
        }
    }
    
    func setLocalNotification(title: String, subtitle: String, body: String, when: Double) {
        let content = UNMutableNotificationContent()
        content.title = title
        content.subtitle = subtitle
        content.body = body
        
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: when, repeats: false)
        let request = UNNotificationRequest.init(identifier: "localNotificatoin", content: content, trigger: trigger)
        UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
        
    }
}

struct LocalNotificationDemoView: View {
    @StateObject var localNotification = LocalNotification()
    @ObservedObject var notificationCenter: NotificationCenter
    var body: some View {
        VStack {
            Button("schedule Notification") {
                localNotification.setLocalNotification(title: "title",
                                                       subtitle: "Subtitle",
                                                       body: "this is body",
                                                       when: 10)
            }
            
            if let dumbData = notificationCenter.dumbData  {
                Text("Old Notification Payload:")
                Text(dumbData.actionIdentifier)
                Text(dumbData.notification.request.content.body)
                Text(dumbData.notification.request.content.title)
                Text(dumbData.notification.request.content.subtitle)
            }
        }
    }
}

extension UIApplicationDelegate {
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        print("Successfully registered for notifications!")
    }

    func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {
        print("Failed to register for notifications: \(error.localizedDescription)")
    }
}

struct PageView: View {
     @State private var selection  = 0
    @State private var localSelectionState = 0
    var body: some View {
        
                     
        TabView(selection: $selection) {
            ForEach(0..<30) { i in
                ZStack {
                    Color.secondary
                    Text("Row: \(i)").foregroundColor(Color(UIColor.systemBackground))
                }.clipShape(RoundedRectangle(cornerRadius: 10.0, style: .continuous))
                //.scaleEffect((selection == i) ? 1.0 :  0.8)
            }
            .padding(.all, 10)
        }.onChange(of: selection, perform: { value in
            withAnimation {
                localSelectionState  = value
            }
        })
        .frame(width: UIScreen.main.bounds.width - 200, height: 200)
        .tabViewStyle(PageTabViewStyle())
        
    }
}

struct PageView_Previews: PreviewProvider {
  static var previews: some View {
    ScrollView {
        LazyHStack {
            PageView()
        }
    }
  }
}
```