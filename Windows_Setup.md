# Windows Setup for Android App Dev

Preinitalization of Gradle project steps for Windows developement try to eliminate manual 
download and install of JDK, Gradle, some Android SDK and setting enivourment variables 
for easier access.

Available methods are [WSL](https://learn.microsoft.com/en-us/windows/wsl/about) (running 
GNU/Linux distribution inside Windows, which allows to proceed with tutorial normally after
setting it up), [Cygwin](https://www.cygwin.com/) and [MSYS2](https://www.msys2.org/).

## Cygwin
!!!TODO TEST!!!


## MSYS2

At the time of writing there is no available Android (Platform) SDK package which makes
this method incomplete. OpenJDK needs to be downloaded manually through curl.

```
MSYS2 disk usage (C:\msys64):
- Clean install: ~330 MB
- Gradle total installed size: ~150 MB
```

### 1. Download and install MSYS2 using [https://www.msys2.org/](https://www.msys2.org/)

### 2. Install Gradle and unzip: `pacman -S unzip gradle`

### 3. Install [OpenJDK](https://openjdk.org/install/):
```sh
mkdir java/
cd java/
# For JDK 11, for other versions check https://jdk.java.net/archive/
# Download Windows version, ELF exec is not supported
curl https://download.java.net/java/ga/jdk11/openjdk-11_windows-x64_bin.zip --output openjdk-11.zip
unzip -q openjdk-11.zip
mkdir /usr/lib/jvm/
cp -r jdk-11 /usr/lib/jvm/java-11-openjdk
```

### 4. Add JDK and Gradle to PATH by appending:
```sh
...
# Add Gradle PATH
export PATH="/usr/share/java/gradle/bin/:$PATH"
# Add Java JDK
export PATH="/usr/lib/jvm/java-11-openjdk/bin/:$PATH"
```
to `~/.bashrc` and run `source ~/.bashrc`.

### 5. TODO check if build is possible