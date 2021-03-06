---
title: Running Latest Code
description: NativeScript Documentation - Running Latest Code
position: 40
slug: latest-code
previous_url: /running-latest
---

# Running the Latest Code

Often when working with open-source projects, one needs functionality that has not yet passed the full release cycle, or even functionality that is not yet fully implemented. We know that many of you are experimenters and want to try the latest and greatest features of NativeScript. That is why we tried to make this process simple and easy to follow. There are two ways to get the latest development code for NativeScript:

- You can get it via npm.
- You can build the source code.

## Getting the latest development version via npm

As an open-source project NativeScript keeps not only its source code but its build infrastructure open. That is why we choose [Travis CI](https://travis-ci.org/) for our nightly builds. Every commit in the master branch of all major NativeScript repos triggers a [Travis CI](https://travis-ci.org/) build which publishes an npm package that can be used directly. Follow those simple steps to get the latest development version of NativeScript:

- Uninstall any existing NativeScript versions:

```Console
npm uninstall -g nativescript
```

- Install the latest development version of NativeScript CLI:

```Console
npm install -g nativescript@next
```

- Edit the package.json file in your project and replace tns-core-modules, tns-android and tns-ios versions with `next`:

```JSON
{
	"description": "NativeScript Application",
	"nativescript": {
		"id": "org.nativescript.MyApp",
		"tns-ios": {
			"version": "next"
		},
		"tns-android": {
			"version": "next"
		}
	},
	"dependencies": {
		"tns-core-modules": "next"
	}
}
```

Instead of editing the package.json file by hand, you could run the following commands:

```Console
tns platform add ios@next
tns platform add android@next
tns plugin add tns-core-modules@next
```

- Run the `npm install` command to update the node modules:

```Console
cd <your-project-folder>
npm install
```

You are now ready to use the latest development version of NativeScript.

## Building the source code

### Reasoning

Building the source code is essential when one wants to contribute to an open source project. The statement is applicable for NativeScript as well. According to the [Contribution Guidelines](https://github.com/NativeScript/NativeScript/blob/master/CONTRIBUTING.md), suggesting a fix involves testing the latest code.

### Behind the curtains of running a NativeScript application

1. `npm install nativescript -g` : Node Package Manager (npm) downloads and installs the [NativeScript CLI](https://www.npmjs.com/package/nativescript).
2. `tns create [AppName]` : The NativeScript CLI downloads the [Hello-World template](https://www.npmjs.com/package/tns-template-hello-world) and unpacks it to a folder named after the app name you choose. At the same time, the CLI installs the [NativeScript cross-platform modules](https://www.npmjs.com/package/tns-core-modules). As a result, your application folder now contains an `app` folder, holding the files of your application ([source code](https://github.com/NativeScript/template-hello-world)) and a `node_modules` folder, having the cross-platform modules ([source code](https://github.com/NativeScript/NativeScript)).
3. `tns platform add android/ios` : The NativeScript CLI downloads the latest SemVer-compatible version of the specified runtime, unpacks it and applies transformations to the native (Android Studio or xCode) project (e.g., changes the project name).
4. `tns run android/ios` : The NativeScript CLI copies the files under the `app` folder to the `platforms/[android/ios]/.../app` folder following a specific logic so that these get used later by a native build tool (*gradle*/*xcode-build*). As a next step, the NativeScript CLI executes compilation, deployment and run commands of *gradle* or *xcode-build*.
5. Any JavaScript code gets executed in a V8 or JavaScriptCore engine and embedded in the NativeScript runtimes. Each call to an actual native object gets marshalled via the runtimes to the underlying platform and vice-versa. The runtimes provide JavaScript handles to the native objects.

### Contents of the [NativeScript](https://github.com/NativeScript/NativeScript) repo

The NativeScript framework is built using TypeScript. For that, one of the build steps is TypeScript compilation, which uses TypeScript declarations of the underlying native objects. These are really large files ([android17.d.ts](https://github.com/NativeScript/NativeScript/blob/master/android17.d.ts) and [ios.d.ts](https://github.com/NativeScript/NativeScript/blob/master/ios.d.ts)). The TypeScript compilation with these two files loaded in memory takes a lot of time. To save development time and have as quick and stable feature output, the NativeScript team decided to keep several important applications inside the same repository so that all of them get compiled in a single pass.

Having said that, each subfolder of the [apps](https://github.com/NativeScript/NativeScript/tree/master/apps) subfolder of the repo represents a single application.

### Building the repo
When the repo gets built, it outputs a bunch of packages (stripping the version- and extension- part of the filename for clarity):
- tns-core-modules : the package, containing the core modules. It gets distributed via [npm](https://www.npmjs.com/package/tns-core-modules).
- tns-sample-* : contains some test/demo applications the team uses internally for testing.
- tns-template-* : has templates that will get used once we have the [template-selection functionality](https://github.com/NativeScript/nativescript-cli/issues/374) implemented in the command-line interface.

The repo gets built via the commands:

```Console
npm install -g grunt-cli
npm install
grunt
```

### Using the latest tns_modules

To use the latest tns_modules, simply:
- Build the repo.
- Navigate to your project folder.
- Delete the `tns-core-modules` folder from the `node_modules` subfolder of your project (i.e., `rm -rf node_modules/tns-core-modules` for Linux or `rd /S /Q node_modules\tns-core-modules`).
- Install the newly built package (`npm install [PATH-TO-NATIVESCRIPT-REPO/bin/dist/tns-core-modules-x.x.x.tgz]`).

### Handling internal breaking changes

It is possible that an internal breaking change gets introduced involving an update to both the runtimes and the modules. An internal breaking change would mean that the public API of the tns_modules does not get affected, but a work in progress change in the runtimes requires a change in the internal code of the tns_modules themselves.

When such a case happens, the [ios](https://github.com/NativeScript/ios-runtime) and [android](https://github.com/NativeScript/android-runtime) runtimes must be built separately and updated via the CLI command of:
`tns platform update android/ios --frameworkPath=[Path-to-Runtime-Package]`

### Building the runtimes

As the NativeScript framework gets distributed via npm, the runtimes are also packed as npm packages. For consistency reasons, the native builds (gradle/xcode-build) are wrapped by grunt builds that do the job.

#### Building the Android runtime

The [android runtime](https://github.com/NativeScript/android-runtime) depends on the [android-metadata-generator](https://github.com/NativeScript/android-metadata-generator).

Provided you have all the dependencies set, the easiest way to have the Android runtime built is to clone the two repos to a single folder so that the two are sibling folders, `cd` into the `android-runtime` folder and run:
```
gradle packar -PwidgetsPath=./widgets.jar
```

The resulting tns-android-x.x.x.tgz package will get created in the `dist` folder.

#### Building the iOS runtime

Follow the instructions on setting up the dependencies for building the [ios runtime](https://github.com/NativeScript/ios-runtime) in the repository README and then run `grunt package`.

The build tns-ios-x.x.x.tgx package will get created in the `dist` folder.
