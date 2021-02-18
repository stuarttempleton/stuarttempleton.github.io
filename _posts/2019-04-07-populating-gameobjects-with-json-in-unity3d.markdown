---
layout: post
title:  "Populating Unity3D GameObjects with JSON"
date:   2019-04-07 11:17:49 -0700
categories: blog
excerpt_separator: <!--more-->
---

My game Vegas Prime Retrograde is built on the idea that we reward the player for exploration with lore and other discoverable things strewn throughout the world. It's a fairly simple task to implement in the game, but managing the ever-growing database of lore was a bit of a chore. I needed something that would let me store a small database of information. Nice-to-haves would be the ability to update that database with information about what the player has encountered. As it turns out, Unity3D has a lot of built in tools for loading external data as JSON, so we'll use those. This is how I did it.
<!--more-->
### The Data Structures

The first thing I needed was something that defined the actual data structure itself. For my use, I needed a few classes: discovery, discoverycategory, discoveryindex

In VPR, discoveries are organized in logical categories. Mostly so they're easier for the player to keep track of what's left to discover. For example, we have a set of discoveries that are about "The Crew" of the player's old ship that suffered some kind of tragedy. A discovery itself is simple. It's just a bit of information for the player, a title, and index information to let us dive in easier later. Discoveries look like this:
{% highlight json %}
{"title":"test", "id":0, "category":0, "body":"body"}
{% endhighlight %}

Categories are similarly simple. They have a name, an index, and an array of discoveries. DiscoveryCategories look a bit like this:
{% highlight json %}
{
	"name":"Corporations and Technologies",
	"id":0,
	"discoveries":[
		{"title":"test", "id":0, "category":0, "body":"body"}]
}
{% endhighlight %}

We pull multiple categories, so our categories are an array. Nothing complex. Put it all together and our category list looks a bit like this:
{% highlight json %}
{
	"categories":[
	{
	"name":"Corporations and Technologies",
	"id":0,
	"discoveries":[
		{"title":"test", "id":0, "category":0, "body":"body"},
		{"title":"test", "id":1, "category":0, "body":"body"},
		{"title":"test", "id":2, "category":0, "body":"body"}
		{"title":"test", "id":3, "category":0, "body":"body"}]
	}

	{
	"name":"The Crew",
	"id":1,
	"discoveries":[
		{"title":"test", "id":0, "category":1, "body":"body"},
		{"title":"test", "id":1, "category":1, "body":"body"},
		{"title":"test", "id":2, "category":1, "body":"body"}
		{"title":"test", "id":3, "category":1, "body":"body"}]
	}]
}
{% endhighlight %}

### From JSON to C# 

Here are the class representations of those, which the manager will use to build the game objects. You can see that we're storing the index so that even out of context, we can track it down. We're also using Unity's built in PlayerPrefs to store information about the player's discovery (we call it retrieval) of the specific nugget of lore.

{% highlight csharp %}
[System.Serializable]
public class discovery
{
	public string title;
	public string body;
	public int id;
	public int category;
	public void retrieve() {
		PlayerPrefs.SetInt ("LORE_" + category + "_" + id, 1);
	}
	public bool retrieved() {
		return PlayerPrefs.GetInt("LORE_" + category + "_" + id, 0) == 1 ? true : false;
	}
	public discoveryindex myIndex() {
		return new discoveryindex(category, id);
	}
}

[System.Serializable]
public class discoverycategory
{
	public string name;
	public discovery[] discoveries;
	public int id;
	public discoverycategory(int _id, string _name, discovery[] _discoveries ) {
		id = _id;
		name = _name;
		discoveries = _discoveries;
	}
}

[System.Serializable]
public class discoveryindex 
{
	public int category;
	public int id;
	public discoveryindex(int _cat, int _id)
	{
		category = _cat;
		id = _id;
	}
}
{% endhighlight %}

### Managing and Populating the Object on Start

Things get a little bit more interesting from here. The DiscoveryManager is responsible for the ephemera around our lore. It's how we load it, save it, and how we tap into. This checks for a valid text asset and then loads it using the JsonUtility.FromJsonOverwrite() feature. This overwrites the data structure you pass it using data provide in the text asset. In this case, the object is loading itself! Also included is the ability to export itself into a file of the same format to be reloaded later. Right now we're only doing this in the editor. This also has some utility functions to help index into the data set, but they're pretty simple. Here's the manager in all its glory.

{% highlight csharp %}
public class discoverymanager : MonoBehaviour {

	public discoverycategory[] categories;
	public float LastLoad = 0.0f;

	public TextAsset Source; //the existing discovery database to use.

	void Start () {
		if (Source) {
			Load (Source.text);
		}
	}

	public void Load (string _data) {
		JsonUtility.FromJsonOverwrite(_data, this);
		LastLoad = Time.deltaTime;
	}

	public discovery getDiscovery( discoveryindex index ) {
		return categories [index.category].discoveries [index.id];
	}

	public void retrieve( int cat, int disco ) {
		categories [cat].discoveries [disco].retrieve ();
	}

	public void Export () {
		
		#if UNITY_EDITOR
		string _data = JsonUtility.ToJson(this, true);
		string filename = string.Format(Application.dataPath + "/discovery_db-{0:yyyy-MM-dd_hh-mm-ss-tt}.json", System.DateTime.Now);
		File.WriteAllText (filename, _data);
		Debug.Log ("File saved as " + filename);
		#endif
	}

}
{% endhighlight %}

So there you have it. That's how you create a gameobject that will populate itself with data from a JSON file. You create the data structure classes, the container gameobject, and the JSON data set. 

### Things that are a bit tricky.

It's really easy to make mistakes in the initial data structure especially, if like our data, yours is built on a few layers of classes. But with a little planning and some initial testing, you can make sure it works. You can also choose to populate this from any text source, including a UnityWebRequest get it from a game server!

The built in JsonUtility has some quirks depending on your version. I know when I was working on this I was having a hard time with escaped double quotes. It felt a little hacky to work around it, but a lot of the JSON utilities out there have quirks. Kind of like csv, I guess. 

Anyway. Thanks for checking this out. It was a lot of fun to solve the problem in a way that works for what we need. For us, something that can be generated by a website later means that we can create a front-end to build the lore that works for our writers. That's a pretty big win. We also use this to define bulk information like weapon data for our hacking black ice, so it's proven to be a pretty useful bit of knowledge.

