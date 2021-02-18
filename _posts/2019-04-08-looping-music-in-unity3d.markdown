---
layout: post
title:  "Looping Music During Game Play in Unity3D"
date:   2019-04-08 11:17:49 -0700
categories: blog
excerpt_separator: <!--more-->
---

Our game needed an audio system that allowed us to take music loops and samples and weave them in and out as the action and intensity increased during game play. The actual need itself wasn't huge. We had a series of "acidized" drum loops and we just wanted to be able to make them respond to player movement and activity. When the player was idle, we wanted to pull back to the mellow loops and when the action increases, well, so should the intensity of the audio.

We wanted to take advantage of Unity3d's built in audio system. It's fairly robust and simple, and provides a scheduler we can easily access in code. Perfect. Here's how I did it.
<!--more-->
### Two Turn-Tables (and an audio clip?)

The foundation of this technique is to mimic a rudimentary DJ. We have two samples that need to play. We don't want to eat too much time swapping clips in and out, so just like a DJ, we use two separate sources. We also have to beat match like a DJ would. So we track what we intend to use as the BPM. Our audio is all 120bpm, so we set it at that. We also track our bar. Ours is 16. We have some basic stuff to help keep track of what we're currently playing and to set a floor for the lowest clip we're willing to play.

{% highlight csharp %}
public AudioClip[] MusicLoops;

public AudioSource[] AudioSources;
public float bpm = 120.0F;
public int numBeatsPerSegment = 16;
public int currentClip = 0;
public int FloorClip = 0; //you can't go lower than FloorClip
public AudioMixerGroup mixerGroup;

private bool running = false;
private double nextEventTime;
private int flip = 0;

public bool PriorityLock = false;
{% endhighlight %}

### Setting Up the AudioSources

The first thing we do, after a few guard statements, is build the two turn-tables and set the sources with our appropriate settings. We're defining mixer groups and turning off looping. We'll handle that ourselves. We also set the "running" flag and set the first cue time.

{% highlight csharp %}
void Start () {
  if ( MusicLoops.Length < 1 ) {
    Destroy(gameObject);
  }
  FloorClip = FloorClip < 0 ? 0 : FloorClip;
  AudioSources = new AudioSource[2];
  AudioSources[0] = gameObject.AddComponent<AudioSource>();
  AudioSources[1] = gameObject.AddComponent<AudioSource>();
  AudioSources[0].playOnAwake = AudioSources[0].loop = false;
  AudioSources[1].playOnAwake = AudioSources[1].loop = false;

  if (mixerGroup) {
    AudioSources[0].outputAudioMixerGroup = mixerGroup;
    AudioSources[1].outputAudioMixerGroup = mixerGroup;
  }
  running = true;

  nextEventTime = AudioSettings.dspTime + 1.0F;
}
{% endhighlight %}
### Making it actually play something.

During the Update() cycle, we check in with our player. If we're not running, we kindly bail. Otherwise, we get right to it. We get the audio clock and check how soon our next cue is coming. If our cue is timely, we add our loop to the AudioSource and then give it a PlaySchedule. Once that's done, we calculate the next play schedule. Rinse and repeat. The selected loop will re-cue at the appropriate time! Yay! It's looping!
{% highlight csharp %}
void Update () {
  if (!running)
    return;
  
  double time = AudioSettings.dspTime;
  if (time + 1.0F > nextEventTime) {
      AudioSources[flip].clip = MusicLoops[currentClip];
      AudioSources[flip].PlayScheduled(nextEventTime);
      nextEventTime += 60.0F / bpm * numBeatsPerSegment;
      flip = 1 - flip;
  }
}
{% endhighlight %}
### But how do you change the loop?

I'm glad you asked! We create a few public helper functions that allow us to set the loop or increment to the next loop in the list. We're basically duplicating an audio seek. We also have a helper to make sure we're not going out of bounds. One of the things I added was a priority lock. I wanted some external systems to have the ability to take control of the loop until it is done. Like a boss fight, for example. It isn't strictly necessary. Once we actually set a loop with SetLoop(), at the next play cue, the Update() function will start looping with the new AudioClip. Pretty simple.
{% highlight csharp %}
public void SetLoopAndLock ( int _newLoop, bool _lock ) {
  PriorityLock = _lock;
  currentClip = CheckLoopBounds ( _newLoop );
}
public void SetLoop ( int _newLoop ) {
  if (PriorityLock) {
    Debug.Log ("Loop rejected due to lock.");
    return;
  }
  currentClip = CheckLoopBounds ( _newLoop );
}
public void NextLoop () {
  SetLoop ( currentClip + 1 );
}
public void PreviousLoop () {
  SetLoop ( currentClip - 1 );
}

private int CheckLoopBounds (  int _newLoop) {
  if ( _newLoop >= MusicLoops.Length ) {
    _newLoop = MusicLoops.Length - 1;
  }
  if ( _newLoop < FloorClip ) {
    _newLoop = FloorClip;
  }
  return _newLoop;
}

public void StartStop () {
  if ( running ) {
    AudioSources[0].Stop();
    AudioSources[1].Stop();
  }
  running = !running;
}
{% endhighlight %}
So, it would look something like this, called from somewhere else.
{% highlight csharp %}
MusicLoopManager _loopManager;
int FancyLoop = 3;

_loopManager = GameObject.Find("MusicLoopManager").GetComponent<MusicLoopManager>();
_loopManager.SetLoop ( 3 );
{% endhighlight %}
### Getting information out of the manager

We also wanted to reuse this code in way that let present play info and control to a Canvas UI, so we created some small helpers that would help us get NowPlaying information as well as the currently playing source and what's up next. Stuff like that.
{% highlight csharp %}
public AudioClip GetPlayingSource () {
  if ( AudioSources[0].isPlaying ) 
    return AudioSources[0].clip;
  if ( AudioSources[1].isPlaying ) 
    return AudioSources[1].clip;

    return null;
}

public bool DecksArePlaying () {
  return ( AudioSources[0].isPlaying || AudioSources[1].isPlaying);
}
{% endhighlight %}
You'd use it like this in a UI or something. Though, probably something less clumsy, I'd hope.
{% highlight csharp %}
MusicLoopManager _loopManager;
public Text Playing;
public Text UpNext;

_loopManager = GameObject.Find("MusicLoopManager").GetComponent<MusicLoopManager>();

Playing.text = _loopManager.GetPlayingSource() ? _loopManager.GetPlayingSource().name : "Not Playing";
UpNext.text = _loopManager.MusicLoops[_looper.currentClip].name;
{% endhighlight %}
### Taking it Further

This met our needs so we stopped here, but I could see this being a framework for a more complex system. More AudioSources,better handling of loops with different BPM and stuff like that. Precomputing that information by analyzing the audio, maybe. The more dynamic it becomes the more you could do with varying sources. 

Thanks for reading!