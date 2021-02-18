---
layout: post
title:  "Basic Collision Detection in Lua"
date:   2019-05-09 11:17:49 -0700
remote_url: "https://git.openformgames.com/stu/luacollisiondetection"
remote_url_action: "Download on Gitlab"
categories: blog
excerpt_separator: <!--more-->
---

I spent some time evaluating using Lua in one of our games. I hadn't spent a large amount of time with the language, so I wanted to implement some rudimentary collision detection code to see how it worked. This might be helpful to you, but it was mostly just for me to get a feel for working in the language in a game development context. This is how I went about it in Lua.

TLDR; I liked it! It can offer a lot of power to the end user if integrated well.
<!--more-->
### Points and Distance

I set up an easy way to work with points and track distance. In come cases, this can be helpful in cutting down on how many collision tests we perform, so I wanted to include it in the kit. This is in a separate file, points.lua.

{% highlight lua %}
local points = {}

-- create the point object
function points.new( _x, _y )
	return {x = _x, y = _y}
end

-- test the distance between 2 points
function points.distance( point1, point2 )
	return math.sqrt((point2.x - point1.x)^2 + (point2.y - point1.y)^2)
end

return points
{% endhighlight %}

### 2D Rects

I set up an easy way to work with rectangles, based losely off of other implementations. I created an AABB collision test. This is basic, so we're assuming axis-aligned. This could be suitable for a number of 2d implmentations. This is in rects.lua.

{% highlight lua %}
local rects = {}

-- create rectangle object
function rects.create( _x, _y, _height, _width )
	return { x = _x, y = _y, height = _height, width = _width }
end

-- test the rect collision, axis-aligned
function rects.collision(rect1, rect2)
	local ax2, bx2, ay2, by2 = rect1.x + rect1.width, rect2.x + rect2.width, rect1.y + rect1.height, rect2.y + rect2.height
	return ax2 > rect2.x and bx2 > rect1.x and ay2 > rect2.y and by2 > rect2.y
end

return rects
{% endhighlight %}

### 2D Circles

I added the usual function to build the circle. Testing circles is fairly easy. To do that we just need to test the radius and the point. We're basically using distance between the points to check it. The contents of circles.lua is this: 

{% highlight lua %}
local circles = {}

-- create circle object
function circles.create( _x, _y, _radius )
	return { x = _x, y = _y, radius = _radius }
end

-- test circle collision using distance 
function circles.collision(circle1, circle2)
	local dx = circle2.x - circle1.x
	local dy = circle2.y - circle1.y
	return math.sqrt(dx^2 + dy^2) <= circle1.radius + circle2.radius
end

return circles
{% endhighlight %}

### 3D Cubes

I set up an easy way to define the cube using the standard information. This test relies on axis alignment as well. To do this, you have to have all 3 axis colliding. If you're missing one, the cube could be above it, below it, or off to the side. So, all 3 means collision. Here's what that code looks like.

{% highlight lua %}
local cubes = {}

function cubes.create(_x, _y, _z, _szX, _szY, _szZ)
	return { x = _x, y = _y, z = _z, sizeX = _szX, sizeY = _szY, sizeZ = _szZ }
end

function cubes.collision( cube1, cube2 )
	if(math.abs(cube1.x - cube2.x) < cube1.sizeX + cube2.sizeX) then
		if(math.abs(cube1.y - cube2.y) < cube1.sizeY + cube2.sizeY) then
			if(math.abs(cube1.z - cube2.z) < cube1.sizeZ + cube2.sizeZ) then
				return true
			end
		end
	end
	return false
end

return cubes
{% endhighlight %}


### Putting It All Together

So, what does this basic implemtation look like? How might we use it? You might use these functions in a really basic game, maybe to do some quick tests for grid-based games. Or, if you're really smart, you'll use these to spring board into better, full featured implementations. Here's how these look in use. This is just a simple way to test the functions. Here you go!

{% highlight lua %}
-- test points
point = require("points")
p1, p2 = point.new(1,1), point.new(2,2)

-- test rects
rect = require("rects")
r1, r2, r3 = rect.create(1,1,3,3), rect.create(2,2,5,5), rect.create(10,10,2,2)

-- test circles
circle = require("circles")
c1, c2, c3 = circle.create(1,1,5), circle.create(2,2,5), circle.create(10,10,2)

-- test cubes
cube = require("cubes")
cube1, cube2, cube3 = cube.create(1,1,1,2,2,2), cube.create(1,2,1,2,2,2), cube.create(10,10,10,2,2,2)

-- test the functions!
print ("Distance: ", point.distance(p1,p2))
print ("Rect collision: ", rect.collision(r1,r2))
print ("Rect collision: ", rect.collision(r1,r3))
print ("Circle collision:", circle.collision(c1,c2))
print ("Circle collision:", circle.collision(c1,c3))
print ("Cube collision: ", cube.collision(cube1,cube2))
print ("Cube collision: ", cube.collision(cube1,cube3))
{% endhighlight %}