---
layout: post
title: "Navigation Component"
date: 2022-05-12 14:07:00 -0000
categories: Android
---

# Navigation Component
## Setup
Gradle:
- Navigation component: https://developer.android.com/guide/navigation/navigation-getting-started#Set-up
- SafeArgs: https://developer.android.com/guide/navigation/navigation-pass-data#Safe-args

Add a `FragmentContainerView` to activity layout:
``` xml
<androidx.fragment.app.FragmentContainerView
    android:id="@+id/nav_host_fragment"
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:defaultNavHost="true"
    app:navGraph="@navigation/nav_graph" />
```

Setup `NavController` in activity's `onCreate()`:
``` kotlin
val navHostFragment =
    supportFragmentManager
    .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
navController = navHostFragment.navController
```

Important: `FragmentContainerView` can't be setup with  `findNavController`. See: [findNavController() fails in onCreate when NavHostFragment is inflated with FragmentContainerView](https://issuetracker.google.com/issues/142847973)

## SafeArgs
SafeArgs don't support types that are not supported formats of Android XML values. Basically, we're stuck with `Int`, `Float`, `Boolean`, `String` as well as their nullables and arrays.

You could still pass other types such as `Double` or custom types via Bundle, though that is suggested to be anti-pattern, as destinations (fragments) should obtain UI data themselves with no more than minimum information such as item IDs.

Navigating to a destination:
``` kotlin
val action = FirstFragmentDirections
    .actionNameFragmentToGreetingFragment(
            "Marco",
            21
    )
)
    findNavController().navigate(action)
}
```

Get NavArgs:
``` kotlin
val args: GreetingFragmentArgs by navArgs()

fun greet() {
    val name = args.name
    val age = args.age
}
```

## Passing arguments to start destination
NavGraph starts fragments/activities for you, that's great in most cases, but in this one you'd want to handle initialization yourself.

1. Remove `app:navGraph` attribute from the `FragmentContainerView`
2. `setGraph(graph: NavGraph!, startDestinationArgs: Bundle?)` is the function designed to solve this exact problem. Just build a bundle from the activity, pass the bundle as an argument, then read the bundle in the fragment:
``` kotlin
this.arguments?.let {
    binding.textview.text = it.getString("key") ?: "null"
}
```

## [Scoping ViewModel to a NavGraph](https://developer.android.com/guide/navigation/navigation-programmatic#share_ui-related_data_between_destinations_with_viewmodel) (or a nested graph)
``` kotlin
val viewModel: SignupFlowViewModel by
    navGraphViewModels(R.id.sign_up_graph)
```

## [Navigation in multi-module apps](https://developer.android.com/guide/navigation/navigation-multi-module)
Include destinations with the `<include>` tag:
``` xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_graph"
    app:startDestination="@id/list_nav_graph">

    <include app:graph="@navigation/list_navigation" />
    <include app:graph="@navigation/favorites_navigation" />
    <include app:graph="@navigation/settings_navigation" />
</navigation>
```