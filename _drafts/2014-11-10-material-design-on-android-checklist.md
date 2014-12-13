---
layout: post
category: 开发
tags: [Android, Material]
title: Material Design on Android Checklist
---

[Originally](http://android-developers.blogspot.com/2014/10/material-design-on-android-checklist.html) posted by [Roman Nurik](https://plus.google.com/+RomanNurik/posts), Design Advocate

## 有形的平面

UI 由平面（一片片「数字纸张」）组成，它们排列在不同的层级，并且会把影子投射在后面的平面。

<div class="figure" style="width:200px">
  <img src="http://4.bp.blogspot.com/-mvS6XxdSUIY/VE6ohdtKOuI/AAAAAAAAA8M/2YTcch1jNL8/s1600/layering.gif">
  <strong>图 1.</strong> Surfaces and layering.
</div>

- **Signature element:** Shadows are used to communicate which surfaces are in front of others, helping focus attention and establish hierarchy. [Read more on depth and layering in UIs](http://www.google.com/design/spec/layout/layout-principles.html#layout-principles-dimensionality).

    > **从代码的角度看：**This is the `android:elevation` and `android:translationZ` attribute in Android 5.0. On earlier versions, shadows are normally provided as PNG assets.

- Shadows and surfaces are used in a consistent and structured way. Each shadow indicates a new surface. Surfaces are created thoughtfully and carefully.
- There are generally between 2 and 10 surfaces on the screen at once; avoid too much layering/nesting of surfaces.
- Scrollable content either scrolls to the edges of the screen or behind another surface that casts a shadow over the content’s surface. Never clip an element against an invisible edge—elements don’t just scroll off into nowhere. Put another way, you rarely scroll the ink on a surface; you scroll the surface itself.

    > **从代码的角度看：**`android:clipToPadding=false` often helps with this when using `ListView` and `ScrollView`.

- Surfaces have simple, single-color backgrounds.

## A Bold, Print-Like Aesthetic

The “digital ink” you draw on those pieces of digital paper is informed by classic print design, with an emphasis on bold use of color and type, contextual imagery, and structured whitespace.

<div class="figure" style="width:250px">
  <img src="http://1.bp.blogspot.com/-3akABr2CL3E/VE6oigZafdI/AAAAAAAAA80/IwVyML5cdjg/s400/themecolors.png">
  <strong>图 2.</strong> Primary and accent colors.
</div>

<div class="figure" style="width:250px">
  <img src="http://2.bp.blogspot.com/-UFfyUWYkt-o/VE6ohENvMnI/AAAAAAAAA88/FwAseGmlIjk/s400/keylines.png">
  <strong>图 3.</strong> Keylines.
</div>

- **Signature element:** Apps use a [primary color and an accent color](http://www.google.com/design/spec/style/color.html#color-ui-color-application) (图 2) to color surface backgrounds and key UI widgets such as text fields and checkboxes. The accent color contrasts very well with the primary color (for example an app can use a dark blue primary color and a neon pink accent color). The accent color is high-contrast and is used to call attention to key UI elements, like a circular floating action button, selected tab strips, or form fields.

    > **从代码的角度看：**Set the `android:colorPrimary` and `android:colorAccent` attributes in your theme (drop the android prefix if using AppCompat).
    >
    > AppCompat automatically colors text fields, checkboxes, and more on pre-L devices.

- **Signature element:** On Android 5.0, the status bar is colored to match the app’s primary color, or the current screen’s content. For full-bleed imagery, the status bar can be translucent.

    > **从代码的角度看：**Set the `android:colorPrimaryDark` or `android:statusBarColor` attribute in your theme (drop the android prefix if using AppCompat) or call `Window.setStatusBarColor`.

- Icons, photos/images, text, and other foreground elements are colored “ink” on their surfaces. They don’t have shadows and don’t use gradients.
- Colors extracted from images can be used to color adjacent UI elements or surfaces.

    > **从代码的角度看：**This can be done using the [Palette](https://developer.android.com/reference/android/support/v7/graphics/Palette.html) support library.

- **Signature element:** Icons in the app follow the [system icon guidelines](http://www.google.com/design/spec/style/icons.html#icons-system-icons), and standard icons use the [material design icon set](http://www.google.com/design/spec/resources/sticker-sheets.html#sticker-sheets-system-icons).
- Photos are generally immersive and full-bleed. For example, for detail screens, run edge-to-edge and [can even appear behind the app bar or status bar](http://www.google.com/design/spec/style/imagery.html#imagery-principles).

    > **从代码的角度看：**The new `Toolbar` widget (and its AppCompat equivalent) can be transparent and placed directly in your layout. For the status bar, check [this Stack Overflow post](http://stackoverflow.com/questions/26440879/how-do-i-use-drawerlayout-to-display-over-the-actionbar-toolbar-and-under-the-st/26440880).

- **Signature element:** Where appropriate, elements like body text, thumbnails, app bar titles, etc. are aligned to [3 keylines](http://www.google.com/design/spec/layout/metrics-and-keylines.html#metrics-and-keylines-keylines-and-spacing) (图 3). On phones, those keylines are 16dp and 72dp from the left edge and 16dp from the right edge of the screen. On tablets those values are 24dp and 80dp.
- UI elements are aligned to and sized according to an 8dp baseline grid. For example, app bars are 56dp tall on phones and 64dp tall on tablets. Padding and margins can take on values like 8dp, 16dp, 24dp, etc. More precise text positioning uses a 4dp grid.

## Authentic Motion

Motion helps communicate what’s happening in the UI, providing visual continuity across app contexts and states. Motion also adds delight using smaller-scale transitions. Motion isn’t employed simply for motion’s sake.

<div class="figure" style="width:200px">
  <img src="http://3.bp.blogspot.com/-dadidlU3muU/VE6og4Ra_BI/AAAAAAAAA8E/uVCWrYMetGI/s400/herotransition.gif">
  <strong>图 4.</strong> "Hero" transitions.
</div>

- In general, UI and content elements don’t just appear or disappear—[they animate into place](http://www.google.com/design/spec/animation/meaningful-transitions.html), either together as a unit, or individually.
- **Signature element:** When touching an item to see its details, there’s a “hero” transition (图 4) that moves and scales the item between its position in the browsing screen and its position in the detail screen.

    > **从代码的角度看：**These are called “[shared element transitions](http://developer.android.com/training/material/animations.html#Transitions)” in the SDK. The support version of `FragmentTransaction` also includes some shared element support.

- **Signature element:** [Ripple effects](http://www.google.com/design/spec/animation/responsive-interaction.html#responsive-interaction-responsive-interaction-examples) originating from where you touched the screen are used to show touch feedback on an item.

    > **从代码的角度看：**The default `android:selectableItemBackground` and `android:selectableItemBackgroundBorderless` have this, or you can use `RippleDrawable` (`<ripple>`) to customize the effect. On pre-5.0 devices, ripples aren’t an expected feature, so defer to the default `android:selectableItemBackground` behavior.

- **Signature element:** UI elements can appear using a circular “reveal” animation.

    > **从代码的角度看：**See [this doc](http://developer.android.com/training/material/animations.html#Reveal) or the `ViewAnimationUtils` class for more.

- **Signature element:** Animations are used in more subtle, delightful ways, such as to convey the [transition between icon states](http://www.google.com/design/spec/animation/delightful-details.html#delightful-details-delightful-details) or text states. For example, a “+” icon can morph into an “x” symbol, or an outlined heart icon can be filled using a paint-bucket fill effect.

    > **从代码的角度看：**Icon transitions can be implemented using `AnimatedStateListDrawable` and its XML counterpart. An example can be found in the [Google I/O app source](https://github.com/google/iosched). There’s also support for [animated vector icons](http://developer.android.com/training/material/animations.html#AnimVector).

- Animations and transitions are fast—generally under 300ms.
- Crossfades are often replaced by translate/slide transitions: vertical slides for descendant navigation and horizontal slides for lateral navigation. For slide transitions, prefer quick acceleration and gentle ease-in deceleration over simple linear moves. See the material design spec on [motion](http://www.google.com/design/spec/animation/authentic-motion.html) for more.

## Adaptive Design (and UI Patterns)

Tangible surfaces, bold graphic design, and meaningful motion work together to bring a consistent experience across any screen, be it phones, tablets, laptops, desktops, TVs, wearables, or even cars. Additionally, the key UI patterns below help establish a consistent character for the app across devices.

<div class="figure" style="width:250px">
  <img src="http://1.bp.blogspot.com/-recwqnhet7E/VE6ogYRbXcI/AAAAAAAAA74/WwtzwD_iJlE/s400/fab.png">
  <strong>图 5.</strong> The floating action button.
</div>

- The app uses [responsive design best practices](http://www.youtube.com/watch?v=Jl3-lzlzOJI#t=11m41s) to ensure screens lay themselves out appropriately on [any screen size](http://www.google.com/design/spec/layout/layout-principles.html#layout-principles-responsive-principles), in any orientation. See the [Tablet App Quality Checklist](http://developer.android.com/distribute/googleplay/quality/tablet.html) for a list of ways to optimize for tablets, and [this blog post](http://android-developers.blogspot.com/2012/11/designing-for-tablets-were-here-to-help.html) for high-level tablet optimization tips.
    - In material design, detail screens are often presented as popups that appear using “hero” transitions (see above).
    - In multi-pane layouts, the app can use multiple toolbars to place actions contextually next to their related content.
- **Signature element:** Where appropriate, the app promotes the key action on a screen using a circular [floating action button](http://www.google.com/design/spec/patterns/promoted-actions.html#promoted-actions-floating-action-button) (FAB). The FAB (图 5) is a circular surface, so it casts a shadow. It is colored with a bright, accent color (see above). It performs a primary action such as send, compose, create, add, or search. It floats in front of other surfaces, and is normally at an 8dp elevation. It frequently appears at the bottom right of the screen, or centered on an edge where two surfaces meet (a seam or a step).

### App bar

- **Signature element:** The app uses a standard Android app bar. The app bar doesn’t have an app icon. Color and typography are used for branding instead. The app bar casts a shadow (or has a shadow cast on it by a surface below and behind it). The app bar normally has a 4dp elevation.

    > **从代码的角度看：**Use the new `Toolbar` widget in Android 5.0 that is placed directly into the activity’s view hierarchy. AppCompat also provides `android.support.v7.widget.Toolbar`, which supports all modern platform versions.

- The app bar might be for example 2 or 3 times taller than the standard height; on scroll, the app bar can smoothly collapse into its normal height.
- The app bar might be [completely transparent in some cases](http://www.google.com/design/spec/style/imagery.html#imagery-principles), with the text and actions overlaying an image behind it. For example, see the [Google Play Newsstand](https://play.google.com/store/apps/details?id=com.google.android.apps.magazines) app.
- App bar titles align to the 2nd keyline (see more info on keylines above)

    > **从代码的角度看：**when using the `Toolbar` widget, use the `android:contentInsetStart` attribute.

- Where appropriate, upon scrolling down, the app bar can scroll off the screen, leaving more vertical space for content. Upon scrolling back up, the app bar should be shown again.

### Tabs

<div class="figure" style="width:250px">
  <img src="http://2.bp.blogspot.com/-nAgdUlQO9iM/VE6oh7paeBI/AAAAAAAAA8c/k1E8Vbc2tOg/s400/tabs.png">
  <strong>图 6.</strong> Tabs with material design.
</div>

- **Signature element:** Tabs follow the newer material design [interactions and styling](http://www.google.com/design/spec/components/tabs.html) (图 6). There are no vertical separators between tabs. If the app uses top-level tabs, tabs are [visually a part of the app bar](http://www.google.com/design/spec/components/tabs.html#tabs-usage); tabs are a part of the app bar’s surface.

    > **从代码的角度看：**See the [SlidingTabsBasic](http://developer.android.com/samples/SlidingTabsBasic/index.html) sample code in the SDK or the [Google I/O app source](https://github.com/google/iosched) (particularly the "My Schedule" section for phones).

- Tabs should support a swipe gesture for moving between them.

    > **从代码的角度看：**All tabs should be swipeable using the `ViewPager` widget, which is available in the support library.

- Selected tabs are [indicated](http://www.google.com/design/spec/components/tabs.html#tabs-specs) by a foreground color change and/or a small strip below the tab text (or icon) colored with an accent color. The tab strip should smoothly slide as you swipe between tabs.

### Navigation drawer

<div class="figure" style="width:200px">
  <img src="http://1.bp.blogspot.com/-Ychs6FLcjGA/VE6og8WgM9I/AAAAAAAAA78/uSFiUm5V8kM/s400/drawerlayering.gif">
  <strong>图 7.</strong> Navigation drawers with material design.
</div>

- **Signature element:** If the app uses a navigation drawer, it follows the newer material design interactions and styling (图 7). The drawer appears in front of the app bar. It also appears semitransparent behind the status bar.

    > **从代码的角度看：**Implement drawers using the `DrawerLayout` widget from the support library, along with the new `Toolbar` widget discussed above. See [this Stack Overflow post](http://stackoverflow.com/questions/26440879/how-do-i-use-drawerlayout-to-display-over-the-actionbar-toolbar-and-under-the-st/26440880) for more.

- **Signature element:** The leftmost icon in the app bar is a [navigation drawer indicator](http://www.google.com/design/spec/layout/structure.html#structure-app-bar); the app icon is not visible in the app bar. Optionally, on earlier versions of the platform, if the app has a drawer, the top-left icon can remain the app icon and narrower drawer indicator, as in Android 4.0.
- The drawer is a standard width: No wider than 320dp on phones and 400dp on tablets, but no narrower than the screen width minus the standard toolbar height (360dp - 56dp = 304dp on the Nexus 5)
- Item heights in the drawer follow the baseline grid: 48dp tall rows, 8dp above list sections and 8dp above and below dividers.
- Text and icons should follow the keylines discussed above.

More and more apps from Google and across the Google Play ecosystem will be updating with material design soon, so expect Winter 2014 to be a big quarter for design on Android. For more designer resources on material design, check out the [DesignBytes series](https://www.youtube.com/playlist?list=PLOU2XLYxmsIJFcNKpAV9B_aQmz2h68fw_). For additional developer resources, check the [Creating Apps with Material Design](http://developer.android.com/training/material/index.html) docs!
