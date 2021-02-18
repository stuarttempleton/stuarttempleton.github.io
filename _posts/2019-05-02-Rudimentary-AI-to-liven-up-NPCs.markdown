---
layout: post
title:  "Using Rudimentary AI to Liven Up NPCs"
date:   2019-05-02 11:17:49 -0700
categories: blog
excerpt_separator: <!--more-->
---

If you've ever followed random NPCs around as they walk their paths, you've probably seen how quickly the illusion falls apart. Most NPCs walk looping paths, rarely doing anything more than creating the illusion of a busy and bustling street. But what if you wanted the NPCs to have more character? What if you wanted to give them a sense of purpose? That's what I set out to do after some interesting feedback from our playtest community. 
<!--more-->
After getting that feedback, I quickly realized my mistake: The NPCs were walking around the world and they weren't *in* the world. I wanted them to be a part of the world around them. I wanted them to use the bus stops. I wanted them to sit on the park benches. Use our Data Terms. Say hello to each other. Pet dogs. Check their phone. Read posters and flyers. 

So, how do we solve it? How do we create organic NPC behavior that kind of works? To start, I created an extremely simple framework that allows me to define needs and take action to meet needs. I call it Mazlow.

### Mazlow

First define the ways you want the NPC to be able to express some minor identity. The Mazlow script uses a simple enum to list the types of needs that an NPC can have and respond to in the world and class to describe each need and to take action. 


{% highlight csharp %}
[System.Serializable]
public enum TYPESOFNEEDS
{
	GREETING,
	USE_ATM,
	SIT_DOWN,
	READ_SIGN,
	CROSSWALK
};

[System.Serializable]
public class need
{
	public TYPESOFNEEDS met;
	public UnityAction action;
	public bool meets ( TYPESOFNEEDS _need ) { 
		return met == _need; 
	}
	public void interactIfMeets(TYPESOFNEEDS _need) { 
		if ( meets(_need) ) { 
			action?.Invoke(); 
		} 
	}
}

public need[] NeedsICanMeet; //the needs I meet for others
public need[] NeedsIHave; //the needs I have.
{% endhighlight %}

Every GameObject in the world that has or meets one or more need has this script and defines all of the various ways can respond to those needs. We're just going to use a simple array for the needs they meet and the needs they have. The idea is that two mazlow scripts can simply compare notes. In the example of the NPC that wants to use the ATM, both the character and the ATM have the script -- The ATM lists "USE_ATM" as a need it can meet and the NPC has "USE_ATM" as a need it has. When they get close enough together, they trigger their interaction functions. 

The NPC will alter its path to intercept the ATM and begin an animation to use the ATM. The ATM probably won't have an interaction function because it just sits there. It's fairly simple. The bulk of the reaction happens in a basic OnTriggerEnter(). It essentially works like this:

* I need THIS. You can meet the need of THIS. - I will ACT.
* I need greet, you can meet the need of greetings - I will greet you.
* I can use Data Terms. You can meet the need of data term - I will use my data term action.

{% highlight csharp %}
void OnTriggerEnter(Collider other)
{
	mazlow OtherNeeds = other.gameObject.GetComponent<mazlow>();
	if (OtherNeeds)
	{
		foreach (need _TheirNeed in OtherNeeds.NeedsICanMeet)
		{
			foreach (need _MyNeed in NeedsIHave)
			{
				_MyNeed.interactIfMeets(_MyNeed.met);
			}
		}
	}
}
{% endhighlight %}

Because we're using UnityActions, you can easily chain them together when you define them or even dynamically. If the NPC picks up an item, it might have its own animations to trigger or might unlock a new set of actions to be taken by the NPC. You can just add those in or remove them.

### Where To Go From Here
This is meant as a simple, fast, and rough way to add a touch of character to your NPCs that path around your world. If you wanted to improve on this, you could add features like action probability to fuzz the rate that actions take place a bit. You could also control those dynamically within the game to express urgency (needing to use the bathroom is a good example.) You could manipulate it by building a feature around tracking emotional state -- are they afraid? Are they seeing something that makes them want to call for help? Are they responding to a call for help? If you were really inclined, you could probably enhance this code by changing the ways that these are triggered. Perhaps not just proximity, but maybe game states. 

Feel free to use this and to make it your own! 