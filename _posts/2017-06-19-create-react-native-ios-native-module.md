---
layout: post
title: "How to create a React Native iOS Native Module"
description: "Learn how to create and distribute and iOS React Native Package from scratch"
categories: [react-native, react, ios]
published: true
---

A few weeks ago I decided to help [@gastoon\_\_\_](https://twitter.com/gastoon___) with the refactor and packaging of a react-native library he had created, [react-native-twilio-video-webrtc](https://github.com/blackuy/react-native-twilio-video-webrtc). Before working with the library, @gastoon___ and I worked on a previous project that involved the development of a cross platform application in React, using [React Native](https://facebook.github.io/react-native/) for the mobile applications. I never had the opportunity before to actually go through the process of packaging a React Native library that involved native components so it was a nice challenge.

I'm going to explain two different "techniques" on how to approach the distribution and installation of the code. I will assume that you are familiar with the basics of iOS development and `cocoapods` as a package manager.

# Library Structure

For a react-native package you will probably structure your code in three different folders, src/lib (js files), ios and android. We are only going to focus on the two first folders.

Our src folder will have our javascript files, this is the code that is going to be required by the react-native application that is using the library. Then we have our ios folder, in that folder our objective-c/swift files are going to sitting. One important thing to understand is that our node package will be holding our js code and the native code as well. This is very handy for users since they don't need to download the code with a different tool, they have all the required code to run the library inside the `node_modules/mylib` folder.

Let's see the different ways of installing the native library on the client application now.

## Manual

If you are creating a library without external dependencies on the native side beside the iOS Framemworks, the manual way is good enough probably. This installation process will let the user install the library by just doing `react-native link your-library` after they have installed the node package. The downside IMO of the manual installation is that if you have an external dependency it will be complicated for the user to add it since the installation will need to include new steps that people not used to iOS development can find confusing.

What we need to do in order to enable the link is create an xcodeproj for our native library in the `ios` folder. We do that by going to Xcode and creating a new `Cocoa Touch Static library` project. Once we have our project created we only need to add the files that are relevant to the ios native library.

By doing that we enable users to install our native code like this:

```shell
npm install --save your-library
react-native link your-library
```

## Cocoapods

If the native library depends on other obj-c/swift libraries it is a good idea to provide a podspec file that describes the library and its dependencies. This is the podspec for the `react-native-twilio-video-webrtc` project.

```ruby
require 'json'

package = JSON.parse(File.read(File.join(__dir__, 'package.json')))

Pod::Spec.new do |s|
  s.name           = 'react-native-twilio-video-webrtc'
  s.version        = package['version']
  s.summary        = package['description']
  s.description    = package['description']
  s.license        = package['license']
  s.author         = package['author']
  s.homepage       = package['homepage']
  s.source         = { git: 'https://github.com/blackuy/react-native-twilio-video-web-rtc', tag: s.version }

  s.requires_arc   = true
  s.platform       = :ios, '8.0'

  s.preserve_paths = 'LICENSE', 'README.md', 'package.json', 'index.js'
  s.source_files   = 'ios/*.{h,m}'

  s.dependency 'React'
  s.dependency 'TwilioVideo', '>= 1.0.1'
end
```  

As you can see we are specifying the source files of the native code (`'ios/*.{h,m}'`) and what external dependencies are required on runtime, in this case, `React` and `TwilioVideo`.

Now we need to create a `Podfile` in the application's ios folder. This file will describe the dependencies that our iOS application has.

Here you can see the Podfile of the Example app (Example/ios/Podfile)[https://github.com/blackuy/react-native-twilio-video-webrtc/blob/master/Example/ios/Podfile] inside `react-native-twilio-video-webrtc`.

```ruby
# Uncomment the next line to define a global platform for your project
source 'https://github.com/CocoaPods/Specs.git'

platform :ios, '9.0'

target 'Example' do
  pod 'Yoga', path: '../node_modules/react-native/ReactCommon/yoga/Yoga.podspec'
  pod 'React', path: '../node_modules/react-native'
  pod 'react-native-twilio-video-webrtc', path: '../../'
end
```

Two important things to notice here. We are specifying the `React` and `Yoga` (required by react-native) as local (filesystem) dependencies. This is done this way since react-native started shipping the native code in the node package. Second, we are doing the exact same thing for our own library (remember we said that our native and js code will be shipped inside our node package ?). We are not pointing to the node_modules folders in this case because the project already hosts the files.

Now that we have everything setuped we can install our code:

```shell
npm install --save your-library
cd ios
pod install
```

Don't forget to start using the xcodeworkspace that cocoapods created instead of using the xcodeproj. If you are not familiar with this please check the cocoapods documentation.

## Carthage ?

Probably their are other ways of installing native dependencies using Carthage (never use it in native iOS dev). If anyone has tried this before please let me know!

# Conclusion

I hope this can be useful for anyone trying to package a react-native library that uses native ios code. Any suggestions or different ways to tackle the problem please let me know!

Enjoy 🎉
