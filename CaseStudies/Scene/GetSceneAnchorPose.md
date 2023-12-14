# Get Scene Anchor Pose

```csharp
using System.Collections.Generic;
using UnityEngine;
using System.Threading.Tasks;
using System.Linq;

public class AnchorManager : MonoBehaviour
{
    /// <summary>
    /// Example function to call the getSceneFloorAnchor function.
    /// </summary>
    public void testAnchor()
    {
        Debug.Log("[DebugLOG] start anchor test");
        _ = getSceneFloorAnchor();
    }

    /// <summary>
    /// Retrieves the scene floor anchor and localizes it in the world.
    /// </summary>
    async Task getSceneFloorAnchor()
    {
        var anchors = new List<OVRAnchor>();
        await OVRAnchor.FetchAnchorsAsync<OVRRoomLayout>(anchors);

        // no rooms - call Space Setup or check Scene permission
        if (anchors.Count == 0)
        {
            Debug.Log("[DebugLOG] no room found");
            return; 
        }
        
        var room = anchors.First();
        if (!room.TryGetComponent(out OVRAnchorContainer container))
            return;

        await container.FetchChildrenAsync(anchors);

        foreach (var roomAnchor in anchors)
        {
            // check classification
            if (!roomAnchor.TryGetComponent(out OVRSemanticLabels labels) ||
                !labels.Labels.Contains(OVRSceneManager.Classification.Floor))
            {
                continue;
            }

            // enable locatable/tracking
            if (!roomAnchor.TryGetComponent(out OVRLocatable locatable))
                continue;
            await locatable.SetEnabledAsync(true);

            // localize the anchor
            locatable.TryGetSceneAnchorPose(out var pose);
            var position = pose.ComputeWorldPosition(Camera.main);
            var rotation = pose.ComputeWorldRotation(Camera.main);
            
            Debug.Log("[DebugLOG] first floor anchor's position: " + position + " and rotation: " + rotation);

            break; // only interested in the first floor anchor
        }
    }
}
```

## Reference

[Access Scene Data with OVRAnchor](https://developer.oculus.com/documentation/unity/unity-scene-ovranchor/)