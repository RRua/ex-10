#  Tópicos de Desenvolvimento de Software

## Objectives
- Sign Release Builds
- Structure git repositories according to standard practices
- Develop CI/CD tasks using GitHub actions


### Exercise 1 - Create Release APK

1. Generate the release APK of your app. You can use either AndroidStudio or cmd-line for that task (./gradlew assembleRelease)

2. Try to install the APK on your device (adb install -r <path-to-apk>). Observe what happens.

3. Generate a keystore and signing key using either [openssl]() or Android Studio(Build->Generate Signed APK->...).

4. Paste the following snippet in your module's build.gradle and analyze it.

```
  android {
    ...
     signingConfigs {

        release {
            storeFile file(RELEASE_STORE_FILE)
            storePassword RELEASE_STORE_PASSWORD
            keyAlias RELEASE_KEY_ALIAS
            keyPassword RELEASE_KEY_PASSWORD
            // Optional, specify signing versions used
            v1SigningEnabled true
            v2SigningEnabled true
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        
        }
    }
}

```

5. Put each one of the variables referred in the build.gradle file in the gradle.properties file:


```
...

RELEASE_STORE_FILE=<path-to-jks-file.jks>
RELEASE_STORE_PASSWORD=<store-pwd>
RELEASE_KEY_ALIAS=<signing-key-alias>
RELEASE_KEY_PASSWORD=<signing-key-pwd>

```

6. Repeat 1. and 2. and confirm that the APK can now be installed on your device.


### Exercise 2 - Structure repository using Gitflow

1. Create the develop branch on your local repository and push it to server.
2. Create a feature branch from the develop branch and implement a simple feature (e.g. add a toast saying "hello" to the main screen).
3. Commit the changes locally and merge the changes with the develop branch.
4. Run tests over the new version of the develop branch and publish the updated version to the remote server.
5. Create a new release branch (e.g 0.0.1) from this new version of the develop branch and publish it to the remote server.
6. Test the new release and create a pull request to the main branch of the repository.
7. Merge the pull request with main.
8. Update the develop branch to math the main branch.


### Exercise 3 - CI/CD using GitHub actions

1. Create the following action on your repository:

```
name: run lint

on: 
  push:
    branches: [main, develop]
  workflow_dispatch:
  
jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
    - name: Build with Gradle
      uses: gradle/gradle-build-action@67421db6bd0bf253fb4bd25b31ebb98943c375e1
      with:
        arguments: lint
    - name: Upload Reports
      uses: actions/upload-artifact@v2
      with:
        name: Test-Reports
        path: app/build/reports/lint*
      if: always()
```

2. Trigger the action and observe its output and associated artifacts.

3. Create a custom action that generates the release apk automatically on your repository when there is a push to the main branch. Use the following skeleton and complete the missing information. 

```

name: Android Relase Gen

on:
  <event>

jobs:
 build:
   runs-on: ubuntu-latest
   steps:
   - name: Checkout code
     uses: actions/checkout@v2
   
   - name: Generate Release APK
     run: ./gradlew <gradle-task>
   
   - name: Sign APK
     uses: r0adkll/sign-android-release@v1
     id: sign_app
     with:
      releaseDirectory: app/build/outputs/apk/release
      signingKeyBase64: ${{ secrets.RELEASE_STORE_FILE }}
      alias: ${{ secrets.RELEASE_KEY_ALIAS }}
      keyStorePassword: ${{ secrets.RELEASE_STORE_PASSWORD }}
      keyPassword: ${{ secrets.RELEASE_KEY_PASSWORD }}
      
   - uses: actions/upload-artifact@master
     with:
      name: release.apk
      path: ${{steps.sign_app.outputs.signedReleaseFile}}
      
   - uses: actions/upload-artifact@master
     with:
      name: mapping.txt
      path: app/build/outputs/mapping/release/mapping.txt

```

Hint: encode the keystore as base 64 and add it to the repository's secrets.Add also all the required secrets mentioned in the workflow file: RELEASE_KEY_ALIAS, RELEASE_STORE_PASSWORD, KEY_STORE_PASSWORD, and RELEASE_KEY_PASSWORD.

``` 
openssl base64 < signing_keystore.jks | tr -d '\n' | tee signing_keystore_base64.txt
```

4. Create a custom action that uses the output of the previous workflow to publish the generated APK to GitHub Releases. Hint: dawidd6/action-download-artifact@v2 + softprops/action-gh-release@v1 actions 

