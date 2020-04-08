---
layout: post
title:  "Using a CharacterAudioManager to Express Personality"
date:   2018-04-02 11:17:49 -0700
categories: blog Unity3d
---

One of the challenges you face as a game developer is creating new and unique ways to expose the narrative to the player. One of the most overlooked ways to do this is through ambient character audio. It's easy to remember the dialog audio, but we can actually add a lot by letting the player *hear* how the character experiences the world. What can you reveal about your character's personality by adding little quips as they try to open locked doors, walking into a smelly ally, or succeed (or fail miserably) at some task?

You can show a lot! You can reveal how connected the character is with the world. Are they generally positive? Hopeful? Pessimistic? Annoyed by humorless robots? Enthralled by adorable animals? Are they self-defeatist? 

What's more, though, is what can the character tell the player? Not to give up? That something is particularly interesting? That you need to get a new key-card? That cold areas are bad for your hydraulics actuators? That they're on the right track? That they made a good decision and have learned from some failure?

A good CharacterAudioManger can drive all of those things. This is what I made for Clara in Vegas Prime Retrograde.

### How Our CharacterAudioManager Works

Let's step through the Vegas Prime Retrograde CharacterAudioManager.cs code and talk about what it's doing!

{% highlight csharp %}
public class CharacterAudioManager : MonoBehaviour {

	public enum CharacterAudioTypes{
		EMPTY,
		DOOR_LOCKED,
		RANDOM_INTERACT,
		ENTER_ALLEY,
		ENTER_LOCKERROOM,
		ENTER_LOUNGE,
		ENTER_STASIS_BAY,
		ENTER_RELEVANT_TO_MISSION,
		ENTER_HIDDEN_AREA,
		CANT_DO_THAT,
		OUT_OF_RANGE,
		EXERTION,
		TAKE_STIMS,
		DRINK_COFFEE

	};

	[System.Serializable]
	public struct CharacterAudioItem 
	{
		public CharacterAudioTypes key;
		public string text;
		public AudioClip audio;
	}

	private CharacterAudioItem EmptyItem;

	public CharacterAudioItem[] CharacterAudioItems;

	public AudioSource _audiosource;

	void Start () {
		EmptyItem.key = CharacterAudioTypes.EMPTY;
		EmptyItem.text = "empty";
	}

{% endhighlight %}    

We're defining the basic audio types that we anticpate needing. We also define the struct that matches audio types to audioclip objects so that we actually have something to play back. Also, as a good citizen of the game industry, we added a text field that we can use for closed-captioning. We have an empty object that we can fill with default settings so that we always have something available.

The meat of this feature is the CharacterAudioItem array. We wanted to make sure and have multiple different audio clips for each key. Our main character Clara won't say the exact same thing every time.

And finally, we initialize it.

{% highlight csharp %}
public CharacterAudioItem Item( CharacterAudioTypes _key) {
	List<CharacterAudioItem> _list = Items(_key);

	return _list[Random.Range( 0, _list.Count)];
}
public List<CharacterAudioItem> Items( CharacterAudioTypes _key) {
	List<CharacterAudioItem> _list = new List<CharacterAudioItem>();

	foreach (CharacterAudioItem item in CharacterAudioItems) {
		if (item.key == _key ) {
			_list.Add(item);
		}
	}
	if (_list.Count < 1) {
		_list.Add(EmptyItem);
	}
	return _list;
}
private bool IsPlayable(CharacterAudioItem _audioitem, bool _interuptable) {
	bool playit = true;
	if (!_interuptable && _audiosource.isPlaying ) {
		List<CharacterAudioItem> _list = Items(_audioitem.key);
		foreach (CharacterAudioItem item in CharacterAudioItems) {
			if ( (_audiosource.clip == item.audio) ) {
				playit = false;
			}
		}

	}
	return playit;
}
{% endhighlight %}    

We need some utility functions to help us out. We want the ability to get a single audio item, but we first need to pull a List of items based on the same key. For example, if we want to play a "door locked" audio when the player tries to open a locked door, we pull all of the possible things Clara can say out loud to the player and then pick a random one. She might say "Nope. Locked." or maybe "Hmmm... it's locked." By using these tools, we can make that really easy. 

We also needed a utility function to know if a specific audio item is playable, especially if it isn't "interruptable." If it's dialog we don't want interrupted, and it is currently playing, then we don't want to trigger it. Let it just play out. If it is currently interruptable, then go for it. Play it.

{% highlight csharp %}

	public void PlayItemFromKey( CharacterAudioTypes _key, bool _interuptable) {
		CharacterAudioItem _item = Item(_key);
		
		if ( IsPlayable(_item, _interuptable) && _item.text != EmptyItem.text ) {
			if ( _interuptable ) {
				_audiosource.PlayOneShot( _item.audio ) ;
			}
			else {
				_audiosource.loop = false;
				_audiosource.clip = _item.audio;
				_audiosource.Play();
			}
		}
	}

	public void PlayItemFromKey( CharacterAudioTypes _key ) {
		PlayItemFromKey(_key, true );
	}
}
{% endhighlight %}

This is the actual entry point for most uses, playing an item from a key. You know you want to use the "door locked" key, so you use this function to request the CharacterAudioManager playback for that key. It does its best to oblige.

### Using The CharacterAudioManager

To make use of this manager, you can simply call the PlayItemFromKey() function. An example of this are our areas of interest. When Clara walks into certain areas, we trigger specific audio. Clara will tell the player what she is experiencing and feeling at that moment, adding depth and context. In the case of our Stasis Bay, which is a cryogenic facility, clara shudders and mentions how cold it is when she walks enters it for the first time. This adds to the ambience and gives context to the visual storytelling as her breath fogs up. Here's the code for using the CharacterAudioManager with location triggers. it shows just how simple it is to pull this kind of storytelling together.

{% highlight csharp %}
public bool PlayOnlyOnce = false;
private bool played = false;
public CharacterAudioManager.CharacterAudioTypes TypeOfAudioToPlay;

void OnTriggerEnter ( Collider col ) {
	if ( col.tag == "Player" ) {
		if ( !played ) {
			GameObject.Find("CharacterAudioManager").GetComponent<CharacterAudioManager>().PlayItemFromKey(TypeOfAudioToPlay);
			played = true;
		}
	}
}
void OnTriggerExit ( Collider col ) {
	if ( col.tag == "Player" ) {
		if ( played & !PlayOnlyOnce) {
			played = false;
		}
	}
}
{% endhighlight %}    

### Where To Go From Here

I think you could get a lot of mileage out of adding playback-related UnityActions. Maybe something to send closed-captioning to the UI in a more elegant way? I'm curious how you would extend it! Feel free to download this and make it your own.

