# Easy MVVM Navigation with Annotations 

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![](https://jitpack.io/v/mproberts/navigator.svg)](https://jitpack.io/#mproberts/navigator)

An code generation system for navigating between activities while keeping your view-models in pure Java.

## Download

Navigator is available via [Jitpack](https://jitpack.io/#mproberts/navigator).

```groovy
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
}
```

```
dependencies {
    compile 'com.github.mproberts.navigator:annotations:-SNAPSHOT'
    annotationProcessor 'com.github.mproberts.navigator:processor:-SNAPSHOT'
}
```

