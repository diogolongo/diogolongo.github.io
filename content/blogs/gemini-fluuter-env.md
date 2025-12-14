---
title: "Flutter env for Google Jules "
date: 2025-12-13T12:00:00-03:00
draft: false
github_link: "https://github.com/gurusabarish/hugo-profile"
author: "Diogo Longo"
params:
    tags:
      - Gemini
      - AI
      - Mobile
      - Development
      - Jules
      - Google
image: /googlejules.png
description: ""
toc: 
---

When you run Flutter inside an AI coding environment (like Google Jules), the biggest pain is usually **missing system deps**, **Flutter SDK not installed**, and **Android SDK not configured**. The fastest way to make it reproducible is a bootstrap script.

Below is the gist script you can embed directly in you solution.

<code>
<script class="gist" src="https://gist.github.com/diogolongo/b58b05785782b40ad883e91d26ee5238.js"></script> 
</code>
---

## How the script works (step-by-step)

### 1) Fail fast
The script starts with `set -e`, so **any command that fails will stop the whole setup**. This prevents partial / inconsistent environments. 

### 2) Install basic Linux dependencies
It runs `apt-get update` and installs a minimal set of tools like `curl`, `git`, `unzip`, `wget`, and `jq`. These cover the basics for fetching SDKs and working with repos.

### 3) Install Flutter SDK (only if missing)
It checks if `/usr/local/flutter` exists. If not, it downloads the **stable Linux tarball** and extracts it into `/usr/local/flutter`. 
### 4) Install Android SDK Command Line Tools (only if missing)
Same idea: it checks `/usr/local/android-sdk`. If missing, it downloads the Android command line tools zip, unpacks it, and places it at:

`/usr/local/android-sdk/cmdline-tools/latest`

### 5) Export environment variables for the current session
It exports `FLUTTER_HOME`, `ANDROID_HOME`, and updates `PATH` so the current shell can run:
- `flutter`
- `sdkmanager`
- `adb` (platform-tools)

This makes the rest of the script work immediately. 

### 6) Accept licenses + install Android packages
It auto-accepts Android SDK licenses and installs:
- `platform-tools`
- `platforms;android-33`
- `build-tools;33.0.2`

This is what you need for Flutter Android builds in headless environments.

### 7) Fix Git safe.directory + permissions for Flutter
Some managed environments complain about “dubious ownership” when tooling touches `/usr/local/flutter`. The script:
- adds `/usr/local/flutter` as a Git safe directory
- changes ownership so the current user can operate on it

Then it runs `flutter doctor` as a sanity check. 

### 8) Persist PATH/ANDROID_HOME in `.bashrc`
Finally, it appends exports to `~/.bashrc` so new terminals keep the same setup:
- adds Flutter to PATH
- sets `ANDROID_HOME`
- adds cmdline-tools and platform-tools to PATH

It prints a reminder to `source ~/.bashrc` (or restart the terminal) to apply. :contentReference[oaicite:8]{index=8}

---

## How to run it in Jules

1. Save the script as `flutter_jules_env.sh`
2. Make it executable:
   ```bash
   chmod +x flutter_jules_env.sh
Run:

````bash
./flutter_jules_env.sh
````
Apply PATH changes (new shell) or:

````bash
source ~/.bashrc
````
That’s it — after this you should be able to run:
````bash
flutter doctor -v
flutter pub get
flutter test
````