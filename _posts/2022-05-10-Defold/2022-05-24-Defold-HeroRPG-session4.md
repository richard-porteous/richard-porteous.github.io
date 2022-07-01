## Lets finish the remaining tasks for HeroRPG

---

Defold tutorial has been put on hold. I will get back to it but I need to get past the lack of useful documentation they provide. Untill then I'll be writing on other Game engines and other topics. This has been left in the blog simply as a reference.

---

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

**NOTE** - *Many things are easy in Defold and many things are well documented - but some things just lack sufficient examples and documentation, so it is very easy to get stuck*

I plan on continuing this in the future. 
+ I managed to use the camera as a library. The reson for this is to get the screen to world co-ordinates that sadly lacks on the standard Defold camera for 2D. Its the only way to get point and click to work when your world and screen coordinates differ.


** Sorry if you are enjoying this. I switch to Godot next. I really like Defold, some things are incredibly easy, but some things lack decent examples even in the documentation, while others things are well covered. Not the best place for a beginner. **
