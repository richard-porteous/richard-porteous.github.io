## Lets finish the remaining tasks for HeroRPG

In the last chapter we got the animation working for the player and spent time trying to understand what we have done.

We have the following still to do
+ the player is always in-front or always behind the mayor when the player passes
+ the player can pass right through the mayor.
+ the mayor is blurry.
+ we can move off the edge of the screen.
+ we can't use point and click

and an extra one
+ can I use shift to make the player faster

The mayor is blurry (and the player has an outline facing certain directions). We can fix this by choosing other images, so I will leave for you to investigate for now.

Instead lets take care of the other things.
+ use shift to go faster

Once again, I'm not focusing on the code, not yet! So we gloss over this, but please read anyway.

in our game.input_binding file we add lshift (left shift) and the Action is ***run***.

So lets fix our ***move.script***.

```
local function play_animation(self, animation)
	if self.current_animation ~= animation then
		self.current_animation = animation
		msg.post("#sprite", "play_animation", { id = animation})
	end
	
	go.set("#sprite", "playback_rate", self.moverate)
end

local function set_animation(self)
	sprite.set_hflip("#sprite", self.vel.x < 0) --true if moving left else false
	if self.vel.y > 0 then
		play_animation(self, hash("up"))
	elseif self.vel.y < 0 then
		play_animation(self, hash("down"))
	elseif self.vel.x ~= 0 then
		play_animation(self, hash("horizontal"))
	else
		play_animation(self, hash("player_01"))
	end
end

function init(self)
	self.movespeed = 150
	self.moverate = 1
	
	msg.post(".", "acquire_input_focus")
	self.vel = vmath.vector3()	
	msg.post("/camera#camerascript", "follow")
end

function update(self, dt)
	
	set_animation(self)
	local pos = go.get_position()
	pos = pos + self.vel * dt
	go.set_position(pos)

	self.vel.x = 0
	self.vel.y = 0
end

function on_input(self, action_id, action)


	if action_id == hash("run") then
		if action.pressed then
			self.moverate = 2
		elseif action.released then
			self.moverate = 1
		end
	end

	if action_id == hash("up") then
		self.vel.y = self.movespeed * self.moverate
	elseif action_id == hash("down") then
		self.vel.y = -self.movespeed * self.moverate
	elseif action_id == hash("left") then
		self.vel.x = -self.movespeed * self.moverate
	elseif action_id == hash("right") then
		self.vel.x = self.movespeed * self.moverate
	end

end


```
We added to our init
+ self.movespeed = 150
+ self.moverate = 1


To our animation (animation is faster if speed increases)

+ go.set("#sprite", "playback_rate", self.moverate)

To the top of our on_input we add:
```
	if action_id == hash("run") then
		if action.pressed then
			self.moverate = 2
		elseif action.released then
			self.moverate = 1
		end
	end
```
And we ***changed*** 

a fixed value velocity to a variable multiplied by the move rate
```
	if action_id == hash("up") then
		self.vel.y = self.movespeed * self.moverate
	elseif action_id == hash("down") then
		self.vel.y = -self.movespeed * self.moverate
	elseif action_id == hash("left") then
		self.vel.x = -self.movespeed * self.moverate
	elseif action_id == hash("right") then
		self.vel.x = self.movespeed * self.moverate
	end
```

***Wait what is self*** as in self.movespeed.

Well it simply means we are using a variable in self (the gameobject this code belongs to). In Lua a variable is created the first time you use it. But it may not have a value which is why we set the value in the init (initialization) function. 

PS. ***local*** means not persistant beyond the block we are in.


### **END OF SESSIONS FOR DEFOLD**

**NOTE** - *Many things are easy in Defold, but some things are just not good ideas in general. I get stuck on this problem for a while. It may be a good idea to keep your point and click logic seperate for now. Use one of the tutorials on thier web-site.*

+ we can't use point and click
+ the camera doesn't follow the player and all the examples assume a stationary camera - should we use script or a 3rd party camera?

We can't actually use our Key code for mouse or touch, and to top it off moving to a ***clicked or touch*** position give you more than 8 possible directions.

So gameplay using a mouse or touch screen is going to be different.

Also if you try writing code to get the position of the mouse click your character will likely move off in a direction that doesn't make sense. For most examples you need to keep the camera stationary and the world needs to map to the camera i.e. 0,0 in the world need to be 0,0 on the camera. This is not what we want.



So to quickly explain - the 2D Orthographic camera in defold needs a little work, and they will get to it someday, but for now they have given us 3 options.

1. We can copy the default.renderer and default.renderer_script to a folder in main and modify the render_render script. We then have to go into the build settings and change the render settings to our new renderer and change our new renderer to use our new renderer_script. The script change is provided [here](https://defold.com/manuals/camera/#converting-mouse-to-world-coordinates), but I found it difficult to find how to use it. The docs are good, just not that good.
2. [Rendercam](https://defold.com/assets/rendercam/) (2D & 3D) by Ross Grams ~ use the bottom link at defold.com/assets/rendercam/
3.  [Ortographic camera](https://defold.com/assets/orthographic/) (2D only) by Björn Ritzl. ~once again use the bottom link at defold.com/assets/orthographic/

Well that is difficult, you may say. Well guess what, it is simpler than it sounds and it gives you a huge advantage that allows you to change the way a game works. But ... that doesn't help you now ... so we will :
+ download the orthographic camera
+ using file explorer or your systems file manager extract it.


For simplicity lets move directly to where the mouse wants.

Ok one more problem, the current code moves us as long as we are pressing the keys, do we want the same? Do we want to move to where we clicked? how do we control the speed if we choose the second way?

Good questions! The fact you asked them means you are thinking about what you are doing.

Click and hold follows what we do currently do. I like click and release better, it gives a little more accuracy.

Take a look at the ***Learn examples*** for ***movement*** on the Defold.com website.
They have *move_to* and *follow* examples. The *move_to* increases speed if the distance is further, not what we want. *follow* is not what we want either. Lets use the ***move_to*** but fix the speed.

***NOTE*** although both these choices would be the wrong ones in a competitive game as it may give one player an advantage. Our game doesn't need to be concered about that, not yet!

** Sorry if you are enjoying this. I switch to Godot next. I really like Defold, some things are incredibly easy, but some things lack decent examples even in the documentation, while others things are well covered. Not the best place for a beginner. **
