Dexter [![Build Status](https://travis-ci.org/Karumi/Dexter.svg?branch=master)](https://travis-ci.org/Karumi/Dexter) [![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.karumi/dexter/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.karumi/dexter) [![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-Dexter-green.svg?style=true)](https://android-arsenal.com/details/1/2804)
======


Dexter is an Android library that simplifies the process of requesting permissions at runtime.

Android Marshmallow includes a new functionality to let users grant or deny permissions when running an app instead of granting them all when installing it. This approach gives the user more control over applications but requires developers to add lots of code to support it.

The official API is heavily coupled with the ``Activity`` class.
Dexter frees your permission code from your activities and lets you write that logic anywhere you want.


Screenshots
-----------

![Demo screenshot][1]

Usage
-----

To start using the library you just need to initialize Dexter with a ``Context``, preferably your ``Application`` as it won't be destroyed during your app lifetime:

```java
public MyApplication extends Application {
	@Override public void onCreate() {
		super.onCreate();
		Dexter.initialize(context);
	}
}
```

Once the library is initialized you can start checking permissions at will. You have two options, you can either check for a single permission or check for multiple permissions at once.

###Single permission 
For each permission, register a ``PermissionListener`` implementation to receive the state of the request:

```java
Dexter.checkPermission(new PermissionListener() {
	@Override public void onPermissionGranted(PermissionGrantedResponse response) {/* ... */}
	@Override public void onPermissionDenied(PermissionDeniedResponse response) {/* ... */}
	@Override public void onPermissionRationaleShouldBeShown(PermissionRequest permission, PermissionToken token) {/* ... */}
}, Manifest.permission.CAMERA);
```

To make your life easier we offer some ``PermissionListener`` implementations to perform recurrent actions:

* ``EmptyPermissionListener`` to make it easier to implement only the methods you want.
* ``DialogOnDeniedPermissionListener`` to show a configurable dialog whenever the user rejects a permission request:

```java
PermissionListener dialogPermissionListener =
	DialogOnDeniedPermissionListener.Builder
		.withContext(context)
		.withTitle("Camera permission")
		.withMessage("Camera permission is needed to take pictures of your cat")
		.withButtonText(android.R.string.ok)
		.withIcon(R.mipmap.my_icon)
		.build();
Dexter.checkPermission(dialogPermissionListener, Manifest.permission.CAMERA);
```

* ``SnackbarOnDeniedPermissionListener`` to show a snackbar message whenever the user rejects a permission request:

```java
PermissionListener snackbarPermissionListener =
	SnackbarOnDeniedPermissionListener.Builder
		.with(rootView, "Camera access is needed to take pictures of your dog")
		.withOpenSettingsButton("Settings")
		.build();
Dexter.checkPermission(snackbarPermissionListener, Manifest.permission.CAMERA);
```

* ``CompositePermissionListener`` to compound multiple listeners into one:

```java
PermissionListener snackbarPermissionListener = /*...*/;
PermissionListener dialogPermissionListener = /*...*/;
Dexter.checkPermission(new CompositePermissionListener(snackbarPermissionListener, dialogPermissionListener, /*...*/), Manifest.permission.CAMERA);
```

###Multiple permissions
If you want to request multiple permissions you just need to do the same but registering an implementation of ``MultiplePermissionsListener``:

```java
Collection<String> permissions = Arrays.asList(
Dexter.checkPermissions(new MultiplePermissionsListener() {
	@Override public void onPermissionsChecked(MultiplePermissionsReport report) {/* ... */}
	@Override public void onPermissionRationaleShouldBeShown(List<PermissionRequest> permissions, PermissionToken token) {/* ... */}
}, Manifest.permission.CAMERA, Manifest.permission.READ_CONTACTS, Manifest.permission.RECORD_AUDIO);
```

The ``MultiplePermissionsReport`` contains all the details of the permission request like the list of denied/granted permissions or utility methods like ``areAllPermissionsGranted`` and ``isAnyPermissionPermanentlyDenied``.

As with the single permission listener, there are also some useful implementations for recurring patterns:

* ``EmptyMultiplePermissionsListener`` to make it easier to implement only the methods you want.
* ``DialogOnAnyDeniedMultiplePermissionsListener`` to show a configurable dialog whenever the user rejects at least one permission:

```java
MultiplePermissionsListener dialogMultiplePermissionsListener =
	DialogOnAnyDeniedMultiplePermissionsListener.Builder
		.withContext(context)
		.withTitle("Camera & audio permission")
		.withMessage("Both camera and audio permission are needed to take pictures of your cat")
		.withButtonText(android.R.string.ok)
		.withIcon(R.mipmap.my_icon)
		.build();
Dexter.checkPermissions(dialogMultiplePermissionsListener, Manifest.permission.CAMERA, Manifest.permission.READ_CONTACTS, Manifest.permission.RECORD_AUDIO);
```

* ``SnackbarOnAnyDeniedMultiplePermissionsListener`` to show a snackbar message whenever the user rejects any of the requested permissions:

```java
MultiplePermissionsListener snackbarMultiplePermissionsListener =
	SnackbarOnAnyDeniedMultiplePermissionsListener.Builder
		.with(rootView, "Camera and audio access is needed to take pictures of your dog")
		.withOpenSettingsButton("Settings")
		.build();
Dexter.checkPermissions(snackbarMultiplePermissionsListener, Manifest.permission.CAMERA, Manifest.permission.READ_CONTACTS, Manifest.permission.RECORD_AUDIO);
```

* ``CompositePermissionListener`` to compound multiple listeners into one:

```java
MultiplePermissionsListener snackbarMultiplePermissionsListener = /*...*/;
MultiplePermissionsListener dialogMultiplePermissionsListener = /*...*/;
Dexter.checkPermissions(new CompositePermissionListener(snackbarMultiplePermissionsListener, dialogMultiplePermissionsListener, /*...*/), Manifest.permission.CAMERA, Manifest.permission.RECORD_AUDIO);
```

** If your application has to support configuration changes based on screen rotation remember to add a call to ``Dexter`` in your Activity ``onCreate`` method as follows:**

```java
  @Override protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.sample_activity);
    Dexter.checkPendingPermissions();
  }
```

**IMPORTANT**: Remember to follow the [Google design guidelines] [2] to make your application as user-friendly as possible.

Add it to your project
----------------------

Include the library in your ``build.gradle``

```groovy
dependencies{
    compile 'com.karumi:dexter:2.0.0'
}
```

or to your ``pom.xml`` if you are using Maven

```xml
<dependency>
    <groupId>com.karumi</groupId>
    <artifactId>dexter</artifactId>
    <version>2.0.0</version>
    <type>aar</type>
</dependency>

```

Do you want to contribute?
--------------------------

Feel free to add any useful feature to the library, we will be glad to improve it with your help.

Keep in mind that your PRs **must** be validated by Travis-CI. Please, run a local build with ``./gradlew checkstyle build`` before submiting your code.


Libraries used in this project
------------------------------

* [Butterknife] [3]

License
-------

    Copyright 2015 Karumi

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

[1]: ./art/sample.gif
[2]: http://www.google.es/design/spec/patterns/permissions.html
[3]: https://github.com/JakeWharton/butterknife
