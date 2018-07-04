# Widget

기존의 OS X나 iOS에서는 custom URL Scheme을 이용하거나 custom pasteboard(번들 seed ID를 동일한경우만 가능) 등을 이용하여 어플리케이션 간에 데이터를 **‘전달’**하는 것만 가능했다.

즉, A라는 앱에서 B라는 앱으로 데이터를 전달한 후 B앱으로 컨텍스트가 전환된 후 그 데이터를 받아서 추가적인 화면을 보여주거나 처리를 할 수는 있었지만, **아예 앱간 컨텍스트 전환 없이 A앱 내부에서 B앱이 가진 기능을 바로 사용하는 것이 불가능**했다.

하지만 이번에 새로운 OS에 “익스텐션(Extensions)” 이라는 이름으로 이것을 가능하게 해주는 기술을 애플에서 공개했다. 앱간에 정보를 주고받으면서 앱 to 앱으로 전환이 일어나는 것 자체가 사용자에게 있어서는 불편이었기 때문에 이러한 익스텐션 기능이 잘 적용되면 사용자 경험 자체가 많이 좋아질 것으로 예상된다.
(사용자에게 있어서는 앱전환이 일어나지 않는것으로 보이지만, 실제로 기술적인 관점에서 봤을때 익스텐션 자체는 실행중인 앱과는 다른 독립적인 프로세스인 것에 주의)





*애플 개발자 문서 참고*

## An App Extension’s Life Cycle

Because an app extension is not an app, its life cycle and environment are different. 

An extension launches when a user chooses it from an app’s UI or from a presented activity view controller. An app that a user employs to choose an app extension is called a *host app*. 

A host app defines the context provided to the extension and kicks off the extension life cycle when it sends a request in response to a user action. 

An extension typically terminates soon after it completes the request it received from the host app.

The system instantiates the app extension identified in the host app’s request and sets up a communication channel between them. The extension displays its view within the context of the host app and then uses the items it received in the host app’s request to perform its task

The user performs or cancels the task in the app extension and dismisses it. In response to this action, the extension completes the host app’s request by immediately performing the user’s task or, if necessary, initiating a background process to perform it. The host app tears down the extension’s view and the user returns to their previous context within the host app. When the extension’s task is finished, whether immediately or later, a result may be returned to the host app.

Shortly after the app extension performs its task (or starts a background session to perform it), the system terminates the extension



### How an App Extension Communicates

An app extension communicates primarily with its host app, and does so in terms reminiscent of transaction processing: There is a request from the host and a response from the extension. Figure 2-2 shows a simplified view of the relationship between a running extension, the host app that launched it, and the containing app.

**Figure 2-2**An app extension communicates directly only with the host app![image: ../Art/simple_communication.png](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Art/simple_communication_2x.png)

There is no direct communication between an app extension and its containing app; typically, the containing app isn’t even running while a contained extension is running. An app extension’s containing app and the host app don’t communicate at all.

In a typical request/response transaction, the system opens an app extension on behalf of a host app, conveying data in an *extension context* provided by the host. The extension displays a user interface, performs some work, and, if appropriate for the extension’s purpose, returns data to the host.

The dotted line in Figure 2-2 represents the limited interaction available between an app extension and its containing app. <u>**A Today widget (and no other app extension type) can ask the system to open its containing app by calling the `openURL:completionHandler:` method of the `NSExtensionContext` class.**</u> As indicated by the Read/Write arrows in Figure 2-3, any app extension and its containing app can access shared data **in a privately defined shared container**. The full vocabulary of communication between an extension, its host app, and its containing app is shown in simple form in Figure 2-3.

**Figure 2-3**An app extension can communicate indirectly with its containing app![image: ../Art/detailed_communication.png](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Art/detailed_communication_2x.png)

>  **NOTE** : Behind the scenes, the system uses interprocess communication to ensure that the host app and an app extension can work together to enable a cohesive experience. In your code, you never have to think about this underlying communication mechanism, because you use the higher-level APIs provided by the extension point and the system.



### Some APIs Are Unavailable to App Extensions

Because of its focused role in the system, an app extension is ineligible to participate in certain activities. An app extension cannot:

- Access a `sharedApplication` object, and so cannot use any of the methods on that object

- Use any API marked in header files with the `NS_EXTENSION_UNAVAILABLE` macro, or similar unavailability macro, or any API in an unavailable framework

  For example, in iOS 8.0, the HealthKit framework and EventKit UI framework are unavailable to app extensions.

- Access the camera or microphone on an iOS device (an iMessage app, unlike other app extensions, does have access to these resources, as long as it correctly configures the [NSCameraUsageDescription](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW24)and [NSMicrophoneUsageDescription](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW25) `Info.plist` keys)

- Perform long-running background tasks

  The specifics of this limitation vary by platform, as described in the extension point chapters in this document.

  (An app extension **can initiate uploads or downloads using an `NSURLSession` object**, with results of those operations reported to the containing app.)

- Receive data using AirDrop

  (An app extension can *send* data using AirDrop in the same way an app does: by employing the `UIActivityViewController` class.)



## Creating an App Extension

When you’re ready to develop an app extension, begin by choosing the extension point that supports the user task you want to facilitate. Use the corresponding Xcode app extension template and enhance the default files with custom code and user interface (UI). After you optimize and test your app extension, you’re ready to distribute it within your containing app.

### Begin Development By Choosing the Right Extension Point

Because each extension point targets a well-defined user scenario, your first job is to choose the extension point that supports the type of functionality you plan to deliver. This choice is an important one because it determines the APIs that are available to you and, in some cases, the ways in which APIs behave.

The extension points supported in iOS and OS X, along with their `Info.plist` extension point identifier keys, are described in the section [NSExtensionPointIdentifier](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/AppExtensionKeys.html#//apple_ref/doc/uid/TP40014212-SW15).

After you choose the extension point that makes sense for your app extension, add a new target to your containing app. The easiest way to add an app extension target is to use an Xcode template that provides a target preconfigured for your extension point.

To add a new target to your Xcode app project, choose File > New > Target. In the sidebar on the left side of the new target dialog, choose Application Extension for iOS or OS X. In the pane on the right side of the dialog, Xcode displays the templates you can choose. For example, Figure 3-1 shows the templates you can use to create an iOS app extension.

**Figure 3-1**Xcode supplies several app extension templates you can use![image: ../Art/add_new_target_2x.png](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Art/add_new_target_2x.png)

After you choose a template and finish adding the target to your project, you should be able to build and run the project even before you customize the extension code. When you build an extension based on an Xcode template, you get an **extension bundle that ends in `.appex`.**

In most cases, you can test the default app extension by enabling it in System Preferences or Settings and then accessing it through another app. For example, you can test an OS X Share extension by opening a webpage in Safari, clicking the Share toolbar button, and choosing your extension in the menu that appears.

### Examine the Default App Extension Template

Each app extension template includes a property list file (that is, an `Info.plist` file), a view controller class, and a default user interface, all of which are defined by the extension point. The default view controller class (or *principal class*) can contain stubs for the extension point methods you should implement.

An app extension target’s `Info.plist` file identifies the extension point and may specify some details about your extension. At a minimum, the file includes the **`NSExtension` key and a dictionary of keys and values that the extension point specifies.** For example, the value of the required **`NSExtensionPointIdentifier`** key is the extension point’s reverse DNS name, such as `com.apple.widget-extension`. Here are some of the other keys and values you may see in your extension’s `NSExtension` dictionary:

- `NSExtensionAttributes`

  A dictionary of extension point–specific attributes, such as `PHSupportedMediaTypes` for a Photo Editing extension.

- `NSExtensionPrincipalClass`

  The name of the principal view controller class created by the template, such as `SharingViewController`. When a host app invokes your extension, the extension point instantiates this class.

- `NSExtensionMainStoryboard` (iOS extensions only)

  The default storyboard file for the extension, usually named `MainInterface`.

In addition to the property list settings, a template may set some capabilities by default. Each extension point can define capabilities that make sense for the type of task the extension point supports. For example, an iOS Document Provider extension includes the `com.apple.security.application-groups` entitlement.

All templates for OS X app extensions include the App Sandbox and `com.apple.security.files.user-selected.read-only` entitlements by default. You might need to define additional capabilities for your extension if it needs to do things like use the network or access the user’s photos or contact information.



### Respond to the Host App’s Request

As you learned in [Understand How an App Extension Works](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionOverview.html#//apple_ref/doc/uid/TP40014214-CH2-SW2), **an app extension opens when a user chooses the extension within a host app and the host app issues a request**. At a high level, your extension receives the request, helps the user perform a task, and completes or cancels the request, according to the user’s action. For example, a Share extension receives a request from a host app and responds by displaying its view. After users compose content in the view, they choose to post the content or cancel the post, and the extension completes or cancels the request accordingly.

When a host app sends a request to an app extension, it specifies an extension context. For many extensions, the most important part of the context is the set of items a user wants to work with while they’re in the extension. For example, the context for an OS X Share extension might include a selection of text that a user wants to post.

As soon as a host app issues its request (typically, by calling the `beginRequestWithExtensionContext:`method), your app extension can use the `extensionContext` property on its principal view controller to get the context. Child view controllers also have access to this property through chaining.

Next, you use the `NSExtensionContext` class to examine the context and get the items within it. Often, it works well to get the context and items in your view controller’s `loadView` method so that you can display the information in your view. To get your extension’s context you can use code like the following:

1. `NSExtensionContext *myExtensionContext = self.extensionContext;`

Of particular interest is the context object’s `inputItems` property, which can contain the items your app extension needs to use. The `inputItems` property contains an array of `NSExtensionItem` objects, each of which contains an item the extension can work on. To get the items from the context object, you might use code like this:

1. `NSArray *inputItems = myExtensionContext.inputItems;`

Each `NSExtensionItem` object contains a number of properties that describe aspects of the item, such as its title, content text, attachments, and user info.

Note that the `attachments` property contains an array of media data that’s associated with the item. For example, in an item associated with a sharing request, the `attachments` property might contain a representation of the webpage a user wants to share.

After users work with the input items (if doing so is part of using the app extension), an app extension typically gives users a choice between completing or canceling the task. Depending on the user’s choice, you call either the `completeRequestReturningItems:completionHandler:` method, optionally returning `NSExtensionItem` objects to the host app, or the `cancelRequestWithError:` method, returning an error code.

IMPORTANTIf your app extension calls the `completeRequestReturningItems:completionHandler:` method, provide a `completionHandler` block to, at minimum, suspend your app extension should the system ask you to. For details, read the documentation for the `completionHandler` block of this method, in *NSExtensionContext Class Reference*.

In iOS, your app extension might need a bit more time to complete a potentially lengthy task, such as uploading content to a website. When this is the case, you can use the `NSURLSession` class to initiate a transfer in the background. Because a background transfer uses a separate process, the transfer can continue, as a low priority task, after your extension completes the host app’s request and gets terminated. To learn more about using `NSURLSession` in your extension, see [Performing Uploads and Downloads](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html#//apple_ref/doc/uid/TP40014214-CH21-SW2).

IMPORTANTAlthough you can set up a background URL upload or download task, other types of background tasks, such as supporting VoIP or playing background audio, are not available to extensions.If you include the `UIBackgroundModes` key in your app extension’s `Info.plist` file, the extension will be rejected by the App Store. (To learn more about this key, see [UIBackgroundModes](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html#//apple_ref/doc/uid/TP40009252-SW22).)

### Optimize Efficiency and Performance

App extensions should feel nimble and lightweight to users. Design your app extension to launch quickly, aiming for well under one second. An extension that launches too slowly is terminated by the system.

Memory limits for running app extensions are significantly lower than the memory limits imposed on a foreground app. On both platforms, the system may aggressively terminate extensions because users want to return to their main goal in the host app. Some extensions may have lower memory limits than others: For example, widgets must be especially efficient because users are likely to have several widgets open at the same time.

Your app extension doesn’t own the main run loop, so it’s crucial that you follow the established rules for good behavior in main run loops. For example, if your extension blocks the main run loop, it can create a bad user experience in another extension or app.

Keep in mind that the GPU is a shared resource in the system. App extensions do not get top priority for shared resources; for example, a Today widget that runs a graphics-intensive game might give users a bad experience. The system is likely to terminate such an extension because of memory pressure. Functionality that makes heavy use of system resources is appropriate for an app, not an app extension.

### Design a Streamlined UI

Most extension points require you to supply at least some custom UI that users see when they open your app extension. An extension’s UI should be simple, restrained, and focused on facilitating a single task. To improve performance and the user’s experience, avoid including extraneous UI that doesn’t support your extension’s main task.

Most Xcode app extension templates provide a placeholder UI that you can use to get started.

Users identify your app extension by its icon and its name. An extension’s icon must be the same as the app icon of its containing app. Using the containing app’s icon helps a user be confident that an extension is in fact provided by the app they installed.

In iOS, a custom Action extension uses a template image version of its containing app’s icon, which you must provide.

iOS Share extensions automatically employ the containing app’s icon. If you provide a separate icon in your Share extension target, Xcode ignores it. For all other app extension types, you must provide an icon that matches the containing app’s icon.

For information on how to add an icon to your app extension, see Creating an Asset Catalog and Adding an App Icon Set or Launch Image Set. For more about icon requirements for iOS app extensions, see “App Extensions” in *iOS Human Interface Guidelines*

An app extension needs a short, recognizable name that includes the name of the containing app, using the pattern *<Containing app name>—<App extension name>*. This makes it easier for users to manage extensions throughout the system. You can, optionally, use the containing app’s name as-is for your extension, in the common case that your containing app provides exactly one extension.

The displayed name of your app extension is provided by the extension target’s `CFBundleDisplayName` value, which you can edit in the extension’s `Info.plist` file. If you don’t provide a value for the `CFBundleDisplayName`key, your extension uses the name of its containing app, as it appears in the `CFBundleName` value.

Make sure you localize the app extension’s name when you provide a localized app extension.

Some app extensions also need short descriptions. For example, an OS X widget displays a description to help users choose the widgets they want to see in the Today view. To provide this text, edit the value of the `widget.description` key in your widget’s `InfoPlist.strings` file.

### Ensure Your iOS App Extension Works on All Devices

You must ensure that your submitted app extension is universal: it must work on iPhone, iPod touch, and iPad. This requirement applies no matter which targeted device family you choose for your containing app. The app extension templates in Xcode are configured correctly for the universal targeted device family.

To declare that your app extension is universal, use the targeted device family build setting in Xcode, specifying the “iPhone/iPad” value.

**To ensure that your app extension is universal**

1. In the Xcode project navigator for your keyboard project, select the project file.

   If the project & targets list in the project editor is hidden, show it. To do this, click the button at the left of the project editor tab bar.

2. In the targets group in the project & targets list, select the target for your app extension.

3. Choose the Build Settings tab in the project editor.

   Ensure that the Basic and Combined buttons are selected, to make it easier for you to locate the settings you need here.

4. In the Deployment group in the project editor, view the Targeted Device Family setting. For both the Debug and Release configuration, the value should be “iPhone/iPad.”

   If you find different values, correct them to be “iPhone/iPad.”

Employ Auto Layout and size classes when designing and building your app extension. Test your app extension to ensure it behaves as you expect it to for all device sizes and orientations. Do this in iOS Simulator, as described in *Simulator User Guide*, and, if possible, also test on physical devices in both orientations.

Remember that even if your containing app targets only the iPad device family, your contained app extension can appear in the context of an iPhone app running in compatibility mode.

IMPORTANTTo pass App Review, you must specify “iPhone/iPad” (sometimes called *universal*) as the targeted device family for your app extension, no matter which targeted device family you choose for your containing app.In a future iOS update, an app extension will run only on devices (or in device compatibility modes) natively supported by the extension’s containing app. For example, an extension provided in an iPad-only containing app will not be visible when using an iPhone app in compatibility mode. To ensure the best user experience possible we recommend your containing app and its app extensions are universal.

### Debug, Profile, and Test Your App Extension

NOTEYou must code sign your containing app and its contained app extensions.All the targets in your Xcode project must be code signed in the same way. For example, during testing you can employ ad hoc code signing or use your developer certificate, but must use the same approach for all the targets in your project. For submission to the App Store, use your distribution certificate for all the targets.

Using Xcode to debug an app extension is a lot like using Xcode to debug any other process, but with one important difference: In your extension scheme’s Run phase, you specify a *host app* as the executable. Upon accessing the extension through that specified host’s UI, the Xcode debugger attaches to the extension.

The scheme in an Xcode app extension template uses the Ask On Launch option for the executable. With this option, each time you build and run your project you’re prompted to pick a host app. If you want to instead specify a particular host to use every time, open the scheme editor and use the Info tab for the app extension scheme’s Run phase.

The steps for attaching the Xcode debugger to your app extension are:

1. Enable the app extension’s scheme by choosing Product > Scheme > `MyExtensionName` or by clicking the scheme pop-up menu in the Xcode toolbar and choosing `MyExtensionName`.

2. Click the Build and Run button to tell Xcode to launch your specified host app.

   The Debug navigator indicates it is waiting for you to invoke the app extension.

3. Invoke the app extension by way of the host app’s UI.

   The Xcode debugger attaches to the extension’s process, sets active breakpoints, and lets the extension execute. At this point, you can use the same Xcode debugging features that you use to debug other processes.

NOTEBefore you build and run your app extension project, ensure the extension’s scheme is selected.If you instead build and run using the *containing app* scheme, Xcode does not attach to your app extension unless you invoke it from the containing app, which is an unusual scenario and might not be what you want.If you access your app extension from a host app different from the one specified in the scheme, the Xcode debugger does not attach to the extension.

In OS X, you need to perform the user step of enabling an app extension before you can access it from a host app for testing and debugging. You enable most extension types by using the Extensions pane of System Preferences. You can also open the Extensions pane by choosing More in the Share or Action menu.

For an OS X Today widget, use the Widget Simulator to test and debug it. (There is no separate step for you to perform in System Preferences to enable the widget.)

For a custom keyboard in iOS, use Settings to enable the app extension (Settings > General > Keyboard > Keyboards).

Xcode registers a built app extension for the duration of the debugging session on OS X. This means that if you want to install the development version of your extension on OS X you need to use the Finder to copy it from the build location to a location such as the Applications folder.

NOTEIn the Xcode debug console logs, an app extension’s binary might be associated with the value of the `CFBundleIdentifier` property, instead of the value of the `CFBundleDisplayName` property.

Because app extensions must be responsive and efficient, it's a good idea to watch the debug gauges in the debug navigator while you're running your extension. The debug gauges show how your extension uses the CPU, memory, and other system resources while it runs. If you see evidence of performance problems, such as an unusual spike in CPU usage, you can use Instruments to profile your extension and identify areas for improvement. You can open Instruments while you’re in a debugging session by clicking Profile in Instruments in any debug gauge report (to view a debug gauge report, click the gauge in the debug area). To learn more about the debug gauges, see Debug Your App; to learn how to use Instruments, see *Instruments User Guide*.

NOTEChoosing Product > Profile in Xcode builds and runs an app extension in Instruments directly. Instruments uses the executable set in the Profile section of the scheme as the host for the extension.

To test an app extension using the Xcode testing framework (that is, the XCTest APIs), write tests that exercise the extension code using your containing app as the host environment. To learn more about testing, see *Testing with Xcode*.

### Distribute the Containing App

You can’t submit an app extension to the App Store unless it’s inside a containing app, and you can’t transfer an extension from one app to another.

To deliver an iOS app extension, you must submit a containing app to the App Store.

To deliver an OS X app extension, it’s recommended that you submit your containing app to the App Store, but it’s not required.

NOTEIf you distribute an OS X app extension outside of the Mac App Store, Gatekeeper prevents the extension from running until the user opens and approves the containing app. Further, if you code sign with a certificate other than your Developer ID, users must explicitly override Gatekeeper to open the containing app to make your extension available.

To pass app review, your containing app must provide functionality to users; it can’t just contain app extensions.





익스텐션, 정확히 말해 익스텐션 컨테이너(Extension Container)는 다음과 같은 특징을 지닌다.

- 앱이 아니다.
- 애플 framework 코드를 통해서만 접근된다.
- App to app IPC (Inter-Process Communication)가 아니다.
- 빌드될때 추가적인 타겟을 통해 따로 빌드되며 설치될때는 앱과 같이 설치, 삭제될때도 앱과 함께 삭제된다. (바이너리 자체도 앱과 독립적이다)
- 실행시에도 앱과는 완전히 다른 독립 프로세스(process)로 실행되기때문에  완전히 다른 주소공간(Isolated address space)를 가지게 된다.

한마디로 **익스텐션**은 애플 프레임웍을 통해서 호출되는 기능들의 집합이라고 표현할 수 있다. 익스텐션의 기능들은 애플프레임웍를 사용해서만 실행 가능하며. 호스트앱(host app)이 다이렉트로  익스텐션을 호출하지 못한다. 실행이 애플 프레임웍에의해 된다는 사실을 제외하면, 앱과 거의 동일한 형태를 가진다. 즉, 유저 인터페이스도 가질 수 있고, 이를위한 ViewController, 리소스 등을 모두 사용가능하다.

Pre-release 된 애플의 문서[1](https://www.letmecompile.com/extensions-for-macos-10-10-ios-8/#fn-381-pre_release)에 나와있는 다음 다이어그램을 보면 좀더 이해가 잘 될것이다.

## Human Interface Guidelines

- 위젯은 extension의 일종으로, 적은 양의 유용한 정보를 실시간으로 보여주거나 어플리케이션의 구체적인 작동을 . 위젯을 Customizing하는 것은 비교적 자유로우며 버튼, 텍스트, 레이아웃, 이미지 등을 조작할 수 있다. 
- 위젯은 어플리케이션을 홈스크린에서 3D 터치 했을 때 appear되는 Quick Action List에 나타나며, 또한 Home Screen이나 Lock Screen에서 왼쪽으로 Swipe했을 때 등장하는 Search screen에 사용자가 임의로 추가할 수 있다. 

**빠르게 훑어볼 수 있는 경험을 제공할 것** : People use widgets to get brief updates and perform very simple tasks, so it’s essential to deliver the right amount of information and interactivity. Wherever possible, provide tasks that can be completed in a **single tap**. Panning and scrolling within widgets isn’t supported.

**내용을 빠르게 보여줄 것** : People spend very little time looking at widgets and shouldn’t need to wait for content to load. **Cache information locally** so you can always show **recent information while getting updates.**

**넉넉한 margin을 제공할 것** Avoid extending content to the edges of a widget. In general, provide a margin of at least a few pixels between each edge and the content. Use the app icon at the top of your widget for alignment guidance. Content tends to work well when lined up with the center of this icon. If your app offers a grid-style layout, make sure you provide sufficient and equal padding between grid items. If possible, limit grids of icons and buttons to four per row.

**Be adaptable.** The width of a widget varies, depending on the device and orientation. The height and information displayed by a widget depends on whether it’s collapsed or expanded (not all widgets support expansion). A collapsed widget is the height of roughly two and a half table rows. An expanded widget is ideally no taller than the height of the screen. The quick action list only shows widgets in a collapsed state. When collapsed, a widget shows essential information that can stand alone. When expanded, a widget shows additional information that enhances the primary information. 

**위젯의 Background를 커스터마이징 하는 것을 피할 것** The light, blurred widget background provided by the system is designed for consistency and legibility. Use it whenever possible. Never use a photo as a background, as it may clash with the Lock and Home screen wallpaper.

**일반적으로 검은 색 계열의 시스템 폰트를 사용할 것** The system font is designed for legibility, and dark colors work well with the standard widget background.

**사람들이 적절하게 어플리케이션으로 이동해서 추가적인 작업을 할 수 있도록 할 것** Your widget should operate independently from your app. However, if people might occasionally need to do more than what your widget offers, make it easy to do so. Don’t include an “**Open App**” button that takes space away from your content. Instead, let people **tap the content itself** to see or edit it in your app. 

**좋은 위젯 이름을 고를 것** An app icon and a title appear above each widget’s content. In general, your widget’s name should match your app’s name. If your app offers multiple widgets, consider using your app name for the primary one and clear, concise names for the others. If you use custom titles, consider prefixing them with your app name. 

**Let people know when authentication adds value.** If your widget provides additional functionality when someone is logged into your app, make sure people know about this. For example, an app that shows upcoming reservations might include a message that says “Sign into the app to view your reservations.” when people aren’t logged in.

**Choose a widget for the quick action list.** If your app has multiple widgets, pick one to appear in the quick action menu that appears when someone applies pressure to your app icon on the Home screen using 3D Touch.



## Developer Guidelines







## Reference

apple developer's guideline : https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/index.html

apple developer's guideline : https://developer.apple.com/app-store/review/guidelines/#extensions

https://minieetea.com/2015/01/archives/2732





미디엄 예제 : https://medium.com/@davidskaarup/add-ios-today-widget-to-your-react-native-app-ed9c9b601cc



알아야 될 것 : https://hackernoon.com/app-extensions-and-today-extensions-widget-in-ios-10-e2d9fd9957a8

구체적 예제 : https://code.tutsplus.com/tutorials/ios-8-creating-a-today-widget--cms-22379

샘플 예제 : http://avilos.codes/mobile/ios-swift/ios-swift-%ED%88%AC%EB%8D%B0%EC%9D%B4-%EC%9D%B5%EC%8A%A4%ED%85%90%EC%85%98today-extension/

http://tapito.tistory.com/450

간단 설명 : http://maskkwon.tistory.com/177

 MacOS 10.10 & iOS 8 새기능 익스텐션(Extensions) 개념 잡기 https://www.letmecompile.com/extensions-for-macos-10-10-ios-8/#fn-381-pre_release