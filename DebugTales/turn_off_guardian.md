# Turn Off Guardian System in Your Apps

simply add the following line into your app's AndroidManifest.xml file:

```xml
<uses-feature android:name="com.oculus.feature.CONTEXTUAL_BOUNDARYLESS_APP" android:required="true" />
```

## Reference

[Meta Community Forum](https://communityforums.atmeta.com/t5/Quest-Development/Disable-guardian-in-passthrough-mode/td-p/965915)