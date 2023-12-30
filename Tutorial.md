# Android App development without Android Studio (2023, Giraffe)

Tutorial resolves setup of Android App development in [Gradle](https://gradle.org/) 
using Kotlin and [Android Jetpack](https://developer.android.com/jetpack/) libraries 
([AndroidX namespace](https://developer.android.com/jetpack/androidx/)) for 
[Compose layout](https://developer.android.com/jetpack/compose/layouts/basics).

Versions of dependency packages, dependency packages itself, methods of implementing them,
content of `*.gradle.kts` configs, project structure may differ from version to version, 
which majorly happened on migration to Kotlin and Compose, abandoning [XML Layouts](https://developer.android.com/develop/ui/views/layout/declaring-layout).
Goal of tutorial is to provide useful, current information coming from own discoveries 
on building app from scratch. 

For now **tutorial doesn't cover test units nor debbuging**, yet.

---

Reason behind creation of this guidance is that Google can't provide proper documentation 
for building project, at the time of writing they even provided [wrong project structure in example (src/ and libs/ should be outside of build/)](https://web.archive.org/web/20231029131315/https://developer.android.com/build#build-files) and vague or incosistent 
configurations due to introduction of Kotlin and Jetpack, outdating previous official tutorials,
or confusing and scattered configs of Gradle at first sight.

Steps and requirements do not originate purely from official sources, but are reconstruction 
of Android Studio's setup, templates and documenatation shown on [developer.android.com](https://developer.android.com/build/dependencies),
Gradle (Kotlin DSL) documentation and experimentation.

To brighten up some doubts I recommend to watch [how to make android app without android studio](https://www.youtube.com/watch?v=qGTUDQfgy3s)
to see (slightly outdated and not recommended) process of requirements and project setup and build of apk.
This video inspired me to provide fresher solution and to seek outside of Android Studio's ecosystem.
After understanding general relations between files and directories, meaning of Gradle's 
config properties, proper dependency implementation and some experimenation one thing is sure:
Android app development without Android Studio *is* very possible (although learning curve
may be high, even higher when trying to execute it on Windows) and after setup of LSP and
own IDE it may be even more pleasant.


## Tool and software requirements

- Java JDK:
    - Required by Gradle to work properly. Versions 11 or above is needed.

- Gradle:
    - Initalization and build tool for app. Deals with configuration via configuration files
    (build.gradle.kts, settings.gradle.kts, gradle.properties, app/build.gradle.kts).

- Proper terminal emulator:
    - Terminal that allows good CLI and access to installed packages without adding them 
    to environment variable manually (which is mostly outside of this tutorial scope). For 
    Windows there is Cygwin or MSYS2 - or pain.
    - See [Windows Setup](./Windows_Setup.md) for steps preceding project initalization
    when developing on Windows.

- [fwcd's Kotlin Language Server](https://github.com/fwcd/kotlin-language-server):
    - Code completion, linting and more for any editor/IDE. Used to write code more elegantly.

- Editor/IDE:
    - Any editor using Language Server Protocol is compatible.
    - Tutorial was tested with Visual Studio Code (OSS) and neovim on Manjaro.

- Kotlin (kotlinc):
    - Not as important for project build (Gradle will install it's own kotlinc if needed).

## Android SDK requirements

Every platform needs to have Android SDKs (`build-tools`, `cmdline-tools`) installed and 
set environment variables ANDROID_HOME and ANDROID_SDK_ROOT.

For Arch-based distributions:

- Every package can be installed through AUR which is recommended.
- `sdkmanager` should be used for checking installed by AUR SDKs:
    - [aur/android-sdk-cmdline-tools-latest](https://aur.archlinux.org/packages/android-sdk-cmdline-tools-latest)
    - [aur/android-platform (latest)](https://aur.archlinux.org/packages/android-platform)
- Installing `android-sdk-cmdline-tools-latest` package also sets ANDROID_HOME and ANDROID_SDK_ROOT variables.
- [Android on wiki.archlinux.org](https://wiki.archlinux.org/title/Android)

For Windows/OS without package manager, dependent on downloading installers through web browser:

- Download and unzip [command-line tools](https://developer.android.com/tools/)
from the bottom of [Android Studio page](https://developer.android.com/studio#command-line-tools-only)
- Put tools in accessable directory, like `C:\Users\user\android-sdk`
- [Create environment variable](https://www.howtogeek.com/787217/how-to-edit-environment-variables-on-windows-10-or-11/)
for ANDROID_HOME and ANDROID_SDK_ROOT both pointing to target directory

Install Android Platform SDK of version 33 or above - lower version won't tolerate Jetpack
library (which use @Composite, see [Jetpack Compose Tutorial](https://developer.android.com/jetpack/compose/tutorial)).

## Initalization

Goal setup contains root Gradle project and one subproject `app`, put together into
[multi-project build](https://docs.gradle.org/current/userguide/multi_project_builds.html#multi_project_builds).
This will cover basic layout generated by Android Studio.

### Project structure
```
## Result project structure ##
# root/                      # Project
#   |---...
#   |---gradle.properties
#   |---build.gradle.kts
#   |---settings.gradle.kts
#   \---app/                 # Module (subproject)
#        |---build.gradle.kts
#        |---build/
#        |---libs/
#        \---src/
#              \---main/     # Source set
#                   |---AndroidManifest.xml
#                   |---kotlin/
#                   |       |---com/example/myapp
#                   |
#                   |---res/
#                        |---drawable/
#                        |---values/
#                        \---...
#     
# Mainly inspired by: 
# https://developer.okta.com/blog/2018/08/10/basic-android-without-an-ide
# and https://developer.android.com/build#build-files
##
```

To initialize above project use steps below. After acknowledging steps and how Gradle, 
its configs and Android SDKs behave, use [Gradle project initalization script automating the process](https://github.com/sqbi-q/android-app-init) 


### 1. Initalize Gradle project

Using basic build type with Kotlin DSL, root directory may be created. 
See: [Build Init Plugin on Gradle documentation](https://docs.gradle.org/current/userguide/build_init_plugin.html)

```sh
gradle init --type basic --dsl kotlin --no-incubating --project-name X
```

Result of above command generates `build.gradle.kts` used to describe plugins potentially used by `app`
and `settings.gradle.kts` used to include modules, name root project and specify repositories 
for plugin management and dependency resolution (see: [Settings file](https://developer.android.com/build#settings-file)
and [The top-level build file](https://developer.android.com/build#top-level)).

Change `build.gradle.kts` to:
```kts
plugins {
    id("com.android.application") version "8.1.0" apply false
    id("org.jetbrains.kotlin.android") version "1.9.10" apply false
}
```
and `settings.gradle.kts` to:
```kts
pluginManagement {
    repositories {
        google()
        gradlePluginPortal()
        mavenCentral()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "yourProjectName"
include(":app")
```
Note that versions of each plugin in `build.gradle.kts` may change, to check proper version for
Android Gradle Plugin (`com.android.application`) see [AGP release notes](https://developer.android.com/build/releases/gradle-plugin#updating-gradle). To check proper version for Kotling Gradle Plugin (`com.jetbrains.kotlin.android`) of see [KGP compatibility table with AGP and Gradle](https://kotlinlang.org/docs/gradle-configure-project.html#apply-the-plugin).

Check [the Gradle settings file and the top-level build file on Understanding the Android build system](https://developer.android.com/build#settings-file)
for more information about properties meaning.

Create `app` subproject directories as shown on [project structure](#project-structure).

Subproject's build.gradle.kts determine implemented (needed for building) depenedencies,
namespace and application ID, target, minimal and compile SDK, app version code and name,
compose-kotlin compiler version, JVM version, build types, and other configurations 
like product flavors. See [The module-level build file](https://developer.android.com/build#module-level)
for sample `build.gradle.kts`.

### 2. Setting app dependencies
### 3. Test build and troubleshooting

---

Check [Android API Levels](https://apilevels.com/) for Android versions compared to its
codename, API level and cumulative usage.

Check [Java versions in Android builds](https://developer.android.com/build/jdks) for more
information on proper versions for Java Development Kit.

Check [Compose to Kotlin Compatibility Map](https://developer.android.com/jetpack/androidx/releases/compose-kotlin)
for compose-kotlin compiler version.

## Running

### 1. Linting and building
### 2. Dependency tree
### 3. ADB and emulating


## Editor setup

After initalization of project structure editor of choice should be connected to Kotlin 
Language Server and at the time of writing [JetBrains won't support LSP](https://discuss.kotlinlang.org/t/any-plan-for-supporting-language-server-protocol/2471/2).

What's left for someone outside IntelliJ IDEA ecosystem is still in-developement [kotlin-language-server](https://github.com/fwcd/kotlin-language-server). 
What's worth of notice is [long time of resolving dependencies](https://github.com/fwcd/kotlin-language-server/issues/510), 
which can take up to ten minutes for fresh project.

### 1. VSCode configuration

With already installed Kotlin Language Server on machine, [`Kotlin` extension](https://open-vsx.org/vscode/item?itemName=fwcd.kotlin) by fwcd should be installed. Decline prompts for automatic download of server and go to settings:

- Provide path to `kotlin-language-server` in `Kotlin > Language Server > Path`
- Check `Kotlin > Language Server > Enabled`

after which server should be working when inside Gradle project.

### 2. Nvim configuration

With already installed Kotlin Language Server on machine, nvim should have:

- Configured `vim-plug` for plugin management inside `$MYVIMRC` 
(see: [How to Manage Plugins in Vim/Neovim](https://smarttech101.com/how-to-manage-plugins-in-vim-neovim-ft-vim-plug/))
- Installed plugin [coc.nvim](https://github.com/neoclide/coc.nvim) for hosting language server.