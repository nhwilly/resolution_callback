# resolution_callback

A Flutter project demonstrating issues with the `localeListResolutionCallback` function.

# Problem 1 - Callback fires multiple times when language changes

When launching the app, this callback is fired once, as per the documentation, and works just fine.

However, when changing language preferences via the device settings panel, upon returning to the app, this callback is fired twice.

This is unnecessary, but when coupled with problem #2, can cause other problems.

# Problem 2 - Callback fires multiple times with incorrect device locales when language changes
If the `locale` property is set manually, the callback fires twice, both on startup *and* when a language preference change is made.

Importantly, the second time it fires the `locales` list only includes the value of the manually set `locale` property and not the list as it does during the first call.

Subsequent alterations to language preferences behave as documented, although they all fire twice.

# Reproducing the problems - Setup
1. Add two locales to the device, `en_US` and `es_US`, since both are supported in the code.
2. Make sure line # 27 is commented out.
3. Problem 1
   1. Restart the app.
   2. Note that `localeListResolutionCallback` fires once and contains the proper values.
   3. Launch the device settings control panel and change the order of the language preferences.
   4. Return to the app
   5. Note that `localeListResolutionCallback` fires twice but still contains the proper values.
4. Problem 2
   1. Uncomment out line #27, effectively setting a locale manually using the `locale` property.
   2. Restart the app.
   3. Note that `localeListResolutionCallback` fires twice but the first time contains two locales and the second time contains only the locale that was assigned to the `locale` property.
   4. Launch the device settings control panel and change the order of the language preferences.
   4. Return to the app
   5. Note that `localeListResolutionCallback` fires twice but still contains the proper values.

## Example of Problem #2 - 4.3
```
I/flutter ( 3367): localeListResolutionCallback fired:
I/flutter ( 3367):   locales: [es_US, en_US]
I/flutter ( 3367):   supportedLocales: [en_US, es_US, es_ES]
I/flutter ( 3367): localeListResolutionCallback fired:
I/flutter ( 3367):   locales: [es_US]
I/flutter ( 3367):   supportedLocales: [en_US, es_US, es_ES]
```
   

# Why this matters
We are building a kiosk app with Flutter.  

The `MaterialApp` is rebuilt when the user changes and, using the user's language preferences and the `locale` property, we can change locales even when the `localeListResolutionCallback` isn't triggered.  This allows us to address a multilingual user base from a single, shared device.

However, we are using the same part of this app on a user's personal device and wish to respond to their changes in locale preferences properly.  This is where problem #2 gets ugly.  Effectively, the second call that's triggered tells us the device only supports one locale.

