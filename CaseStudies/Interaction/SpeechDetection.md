# Automatic Speech Detection

Update Date: 2023-12-26

## Step 1: Setting Up the Script

Create a new C# script called `VoiceDetector` and attach it to a GameObject in your scene. This script will be responsible for handling the microphone input and detecting voice activity.

## Step 2: Defining Variables

Open the `VoiceDetector` script, and let's start by defining some variables that we will need:

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class VoiceDetector : MonoBehaviour
{
    public float detectionThreshold = 0.02f;
    public float silenceThreshold = 2.0f;
    private AudioClip detectionClip;
    private List<float> samples = new List<float>();
    private int detectionFrequency = 16000;
    private int detectionLengthSeconds = 5;
    private float lastSoundTime = -1f;
    private bool hasStartedSpeaking = false;
    private int firstSoundPosition = 0;
    private int lastSoundPosition = 0;
}
```

Here, we have:

- `detectionThreshold`: The minimum volume level to consider as a voice detected.
- `silenceThreshold`: The duration of silence to consider that the user has stopped speaking.
- `detectionClip`: The audio clip used to record and analyze the microphone input.
- `samples`: A list to store audio samples from the microphone.
- `detectionFrequency`: The frequency at which the microphone records audio.
- `detectionLengthSeconds`: The duration of the recording buffer in seconds.
- `lastSoundTime`: The time at which the last voice was detected.
- `hasStartedSpeaking`: A flag to determine if the user has started speaking.
- `firstSoundPosition` and `lastSoundPosition`: Variables to mark the positions in the `samples` list where the voice was first and last detected.

## Step 3: Starting the Microphone

In the `Start` method, we will begin recording audio from the microphone:

```csharp
void Start()
{
    StartCoroutine(DetectSpeech());
}
```

## Step 4: Detecting Speech

Now, let's implement the `DetectSpeech` coroutine, which will be responsible for processing the microphone input:

```csharp
IEnumerator DetectSpeech()
{
    int sampleWindow = detectionFrequency * detectionLengthSeconds;
    detectionClip = Microphone.Start(null, true, detectionLengthSeconds, detectionFrequency);
    yield return null; // Wait until the microphone has started.

    while (true)
    {
        if (Microphone.GetPosition(null) <= 0)
        {
            yield return new WaitForSeconds(0.1f); // Wait a bit before trying again.
            continue;
        }

        float[] currentSample = new float[sampleWindow];
        detectionClip.GetData(currentSample, 0);
        ProcessAudioData(currentSample);

        yield return new WaitForSeconds(detectionLengthSeconds);
    }
}
```

## Step 5: Processing Audio Data

The `ProcessAudioData` method will analyze the audio samples and detect if the user has started or stopped speaking:

```csharp
private void ProcessAudioData(float[] newSample)
{
    for (int i = 0; i < newSample.Length; i++)
    {
        if (Mathf.Abs(newSample[i]) > detectionThreshold)
        {
            if (!hasStartedSpeaking)
            {
                hasStartedSpeaking = true;
                firstSoundPosition = samples.Count + i;
                // Log: User has started speaking.
                lastSoundTime = Time.time;
            }
            lastSoundPosition = samples.Count + i;
            lastSoundTime = Time.time;
        }
    }

    samples.AddRange(newSample);

    if (hasStartedSpeaking && Time.time - lastSoundTime > silenceThreshold)
    {
        hasStartedSpeaking = false;
        // Log: User has stopped speaking.
        samples.Clear();
        firstSoundPosition = 0;
        lastSoundPosition = 0;
    }
}
```

In this method, we iterate through the new samples and check if their volume exceeds the `detectionThreshold`. If so, we mark the start and end positions of the detected voice. If the user stops speaking and the silence exceeds our `silenceThreshold`, we clear the samples and reset the positions.

## Conclusion

You now have a basic voice detection system that continuously listens for user input and detects when the user starts and stops speaking. This foundational system can be extended to integrate with speech recognition libraries, trigger events in your game, or record audio clips when needed.

Remember to test your application and adjust the `detectionThreshold` and `silenceThreshold` to suit your needs. Happy coding!