# Xam.Plugins.OnDeviceCustomVision

The [Azure Custom Vision service](https://customvision.ai) is able to create models that can be exported as CoreML or Tensorflow models to do image classification.

This plugin makes it easy to download and use these models offline from inside your mobile app, using CoreML on iOS or Tensorflow on Android. These models can then be called from a .NET standard library, using something like Xam.Plugins.Media to take photos for classification.

#### Setup

* Available on NuGet: https://www.nuget.org/packages/Xam.Plugins.OnDeviceCustomVision/ [![NuGet](https://img.shields.io/nuget/v/Xam.Plugins.OnDeviceCustomVision.svg?label=NuGet)](https://www.nuget.org/packages/Xam.Plugins.OnDeviceCustomVision/)
* Install into your .NET Standard project and iOS and Android client projects.

#### Platform Support

|Platform|Version|
| ------------------- | :------------------: |
|Xamarin.iOS|iOS 11+|
|Xamarin.Android|API 21+|

#### Usage

Before you can use this API, you need to initialise it with the model file downloaded from CustomVision. Trying to classify an image without calling `Init` will result in a `ImageClassifierException` being thrown.

##### iOS

Download the Core ML model from Custom Vision. 

###### Pre-compiled models

Models can be compiled before beiong used, or compiled on the device. To use a pre-compiled model, compile the downloaded model using:

```bash
xcrun coremlcompiler compile <model_file_name>.mlmodel <model_name>.mlmodelc
```

This will spit out a folder called `<model_name>.mlmodelc` containing a number of files. Add this entire folder to the `Resources` folder in your iOS app. Once this has been added, add a call to `Init` to your app delegate, passing in the name of your compiled model without the extension (i.e. the name of the model folder __without__ `mlmodelc`) and the type of model downloaded from the custom vision service:

```cs
public override bool FinishedLaunching(UIApplication uiApplication, NSDictionary launchOptions)
{
   ...
   CrossImageClassifier.Current.Init("<model_name>", ModelType.General);
   return base.FinishedLaunching(uiApplication, launchOptions);
}
```

###### Uncompiled models

Add the downloaded model, called `<model_name>.mlmodel`, to the `Resources` folder in your iOS app.Once this has been added, add a call to `Init` to your app delegate, passing in the name of your model without the extension (i.e. the name of the model folder __without__ `mlmodel`) and the type of model downloaded from the custom vision service:

```cs
public override bool FinishedLaunching(UIApplication uiApplication, NSDictionary launchOptions)
{
   ...
   CrossImageClassifier.Current.Init("<model_name>", ModelType.General);
   return base.FinishedLaunching(uiApplication, launchOptions);
}
```

The call to `Init` will attempt to compile the model, throwning a `ImageClassifierException` if the compile fails.

##### Android

Download the tensorflow model from Custom Vision. This will be a folder containing two files.

* `labels.txt`
* `model.pb`

Add both these files to the `Assets` folder in your Android app. Once these are added, add a call to `Init` to your main activity passing in the name of the model file and the type of model downloaded from the custom vision service:

```cs
protected override void OnCreate(Bundle savedInstanceState)
{
   ...
   CrossImageClassifier.Current.Init("model.pb", ModelType.General);
}
```

Note - the labels file must be present and called `labels.txt`.

##### Calling this from your .NET Standard library

To classify an image, call:

```cs
var tags = await CrossImageClassifier.Current.ClassifyImage(stream);
```

Passing in an image as a stream. You can use a library like [Xam.Plugins.Media](https://github.com/jamesmontemagno/MediaPlugin) to get an image as a stream from the camera or image library.

This will return a list of `ImageClassification` instances, one per tag in the model with the probabilty that the image matches that tag. Probabilities are doubles in the range of 0 - 1, with 1 being 100% probability that the image matches the tag. To find the most likely classification use:

```cs
tags.OrderByDescending(t => t.Probability)
    .First().Tag;
```

##### Using with an IoC container

`CrossImageClassifier.Current` returns an instance of the `IImageClassifier` interface, and this can be stored inside your IoC container and injected where required.
