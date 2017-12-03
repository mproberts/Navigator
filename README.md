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

## Getting Started

What we want to end up with is the simplest possible `Activity`. Our activities should only need to tell us what we want to use as our layout and what to bind to our layout. It's a simple mediator between our view and our view-model, taking the context information from the intent which spawned the activity and fetching the appropriate view-model.

What we **will** end up with using Navigator is this:

#### <code>ProfileActivity.java</code>

```java
public class ProfileActivity extends AndroidBindingProfileActivity {

    @Override
    protected int getLayoutResource() {
        // indicate which layout to inflate and bind to the view-model
        return R.layout.activity_profile;
    }

    @Override
    public ProfileViewModel navigateToProfile(UserId userId) {
        // provide an instance of the view-model to be bound using the
        // supplied context from the intent, and any dependencies from
        // your graph
        return getDomainComponent()
                .plus(new ProfileModule(userId))
                .viewModel();
    }
}
```

The activity is backed by a simple `ProfileViewModel`. It is a pure-Java interface using RxJava 2. The interface has one addition, a `Navigator` inner-interface. This interface will be consumed by other view-models who want to navigate **to** the profile.

#### <code>ProfileViewModel.java</code>
```java
public interface ProfileViewModel {
    // to be used by another view-model to "navigate" to this view-model
    interface Navigator {
        void navigateToProfile(UserId userId);
    }

    Flowable<Optional<String>> displayName();

    Flowable<Optional<String>> username();

    Flowable<Optional<String>> profilePhoto();

    Flowable<Boolean> isPremium();
}
```

One such view-model is the HomeViewModel. The default implementation of the `HomeViewModel` needs to know how to navigate to the `ProfileViewModel` so it takes an instance of the `ProfileViewModel.Navigator` interface as a constructor parameter. So far, we're still in a pure-Java paradise, no fragmentation here.

#### <code>DefaultHomeViewModel.java</code>

Implementations of other view-models which need to navigate to the profile can take an instance of the `ProfileViewModel`'s `Navigator` type as a dependency. This dependency will resolve to the code-genererated `AndroidBindingNavigator` instance. You can resolve this dependency using direct constructor injection or use a dependency injection framework like Dagger.

```java
public class DefaultHomeViewModel implements HomeViewModel {
    private ProfileViewModel.Navigator profileNavigator;
    private UserId currentUserId;// = ...;

    public DefaultHomeViewModel(ProfileViewModel.Navigator profileNavigator) {
        this.profileNavigator = profileNavigator;
    }

    @Override
    public void onOpenProfileTapped() {
        profileNavigator.navigateToProfile(currentUserId);
    }
}
```

Moving to the Android-layer, we need to tell the navigation system that when someone wants to navigate to the `ProfileViewModel`, what they want to do is show the `ProfileActivity` supplying the context of the `UserId`.

#### <code>Navigator.java</code>
```java
@NavigationSource(baseActivity = DemoActivity.class)
public abstract class Navigator implements ProfileViewModel.Navigator, SearchViewModel.Navigator, HomeViewModel.Navigator {

    // Navigate to the ProfileActivity bundling a UserId
    @Override
    @ActivityNavigation(ProfileActivity.class)
    public abstract void navigateToProfile(UserId userId);

    @Override
    @ActivityNavigation(MainActivity.class)
    public abstract void navigateToHome();

    @Override
    @ActivityNavigation(SearchActivity.class)
    public abstract void navigateToSearch();
}
```

We do that by creating a `Navigator` abstract class (or whatever you decide to call it). The `@NavigationSource(...)` annotation signifies that this will generate a navigator class which will marshal data to your activities. The `baseActivity` parameter indicates which Activity type all generated activities will inherit from.

Each distinct Activity on the navigator will create a new base Activity. The implementation of `ProfileActivity` above extended from `AndroidBindingProfileActivity`, that is generated by the Navigator processor. The processor will generate code to automatically marshal the arguments from the navigation call through the `Intent` extras and invoke the correct target method on the Activity, binding the resulting view-model to the layout. A note of caution, only types which are serializable or fit the standard Bundle extra methods are allowed as parameters of the navigation methods. If you are adding dependencies to more complex tasks you likely want to inject those from your dependency graph via constructor arguments to the actual implementations of the view-model classes.

