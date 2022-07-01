## Flip book animation + an explantion

---

Defold tutorial has been put on hold. I will get back to it but I need to get past the lack of useful documentation they provide. Untill then I'll be writing on other Game engines and other topics. This has been left in the blog simply as a reference.

---

+ Lets get the player animated.
+ then I'll expain what we have done so far.

OK last chaper we added an animation to our player atlas

![](/img/session3-images/player_animation_atlas.png)

We forgot to change the animation speed, lets do that quickly.

![](/img/session3-images/frame_rate_for_player.png)

Open the player.atlas and select the animation we created. Set the Fps (frames-per-second) to 8. It was 60 and which would make our animation too fast.


We will make a modification to our scripts and then test the changes.
+ We add a *play_animation* function and a *set_animation* function above the code that will call it. 
+ At the top of the function *update(self, dt)* we call the animation code using *set_animation(self)* at that code chooses which animation to play.
+ ~= reads as "is not equal to"

### **move.script**
```
-- Plays the animation within input parameters, unless the same animation is already playing
local function play_animation(self, animation)
	if self.current_animation ~= animation then
		self.current_animation = animation
		msg.post("#sprite", "play_animation", { id = animation })
	end
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
	msg.post(".", "acquire_input_focus") -- <1>
	self.vel = vmath.vector3() -- <2>	
	msg.post("/camera#camerascript", "follow")
end

function update(self, dt)
	
	set_animation(self)
	local pos = go.get_position() -- <3>
	pos = pos + self.vel * dt -- <4>
	go.set_position(pos) -- <5>

	self.vel.x = 0 -- <6>
	self.vel.y = 0
end

function on_input(self, action_id, action)
	if action_id == hash("up") then
		self.vel.y = 150 -- <7>
	elseif action_id == hash("down") then
		self.vel.y = -150
	elseif action_id == hash("left") then
		self.vel.x = -150 -- <8>
	elseif action_id == hash("right") then
		self.vel.x = 150
	end
end

```

Ok press F5 you see the player animate when you press the arrow keys to walk.

### **OK, Let me explain!**

We still have some more to do, but first I need to spend a few minutes just going over some basics.
Read this and if you don't understand, its ok. As you go along you recall some, you will know where to go for the answer.

If you read and understood ***core concepts*** from the manual great, if you read and grasped some basic concepts about ***addressing*** even better. If you read up on ***Lua*** perfect. If not, not to worry - not yet anyway.

### HERE AGAIN ***Core Concepts*** (brief)

+ Collection.
    + A collection is a file used to structure your game. It holds **game-objects and other collections**. They are typically used to structure game levels, groups of enemies or characters built out of several game objects.
    Our main scene was a collection.
+ Game object
    + A game object is a container with an id, position, rotation and scale. It can hold other **collections**, **game-objects**, and **components**. They are mostly used to create the "things" in a game.
    The player, NPC, buldings, street,  sidewalk and background were game-objects.  
+ Component
    + Components are entities that are put in game object to give them **visual, audible and/or logic representation** in the game (see, hear, do). They are typically used to create character sprites, script files, add sound effects or add particle effects. The sprites are components. In the remaining sessions we will see a few more components.

Read core concepts in the manual for more detail, now is a good time to do it.

### ***Addressing***

When the game engine needs to 'talk' to a game-object or component it will use its address. The address is the full name and path of the object. A # is used to specify a component in the address.

Think of it like telephone numbers. The first part is the country code, followed by an area code, followed by the number that identifies the actual phone. Or a house address when you write a letter. You don't always need country and area code when calling/writing from the same area.

So when an object is called by another object in the same collection, then we dont need to use more than the object-name and if its a component then object-name#component-name. This is called ***relative*** addressing.

If we need to 'talk' to an object outside our collection we would use the ***absolute*** address. In our game the top level collection is main. So we could use main/object#component or /object#component if we are in the same top collection. (It is possible to have other collections).

Please read addressing - this is a complicated subject for a beginner so you may find yourself having to read it several times for the first few weeks/months of using defold. Photographic memory anyone?

The addresses we used are usually in a string. a string can hold text data. The addresses we get back from the engine are always hashed. Hash is **not #** in this case. Its more of a unique name turned into a number. If we want to compare our address to an address used by the system, we have to use ***hash("/path/to/the/object")*** i.e. the *absolute* address hashed.

### ***Messages***

When using code you will find some functions being called by the engine or being called by a script in another game object. An example message example is the code ***msg.post("/camera#camerascript", "follow")*** found in the player script that tells the camera-script ***"follow"*** and the camera also get the imformation who sent the message. The camera in turn recieves the message with this function

```
function on_message(self, message_id, message, sender)
	if message_id == hash("follow") then
```

  Defold uses messages. We simply need the address of the object containing the code and we can 'talk' to that object by using messages. We can also pass addtioninal information.

### ***Lua***

As a beginner, I'm going to say - read it like english, and when stuck look at the back of the book for links where you can find more detail.

Variables contain values we can use in our code i.e.
*pos = pos + self.vel * dt* <br/> where pos, vel, and dt are variables.

You will see ***functions*** - these are blocks of code that can be called from another part of code, they and other blocks are terminated with the word ***end***. Each block of code will have its own ***end***.

Our code above is fairly simple to explain
+ local function play_animation(self, animation). this code block checks the variable called animation to see what animation is currently playing and request the sprite to play the animation if it is not currently playing.
    + *if self.current_animation ~= animation then ... end* i.e. if not the same then we can do the stuff in the block.
    + *self.current_animation = animation* we make them the same for the next time we check. i.e. we set the value of *current_animation* variable to be the same value as the *animation* variable.
    + *msg.post("#sprite", "play_animation", { id = animation })* we send a message to the sprite component telling it to play the animation with the same name as the name stored in the *animation* variable.
+ *local function set_animation(self)* simply checks by our current speed and direction (velocity) to decide what animation to play.
+ *function init(self)*. This is the function that gets called by the game engine when the object is created. We post a message to get input focus, we post a message to tell the camera to follow us, and we set the variable to be a vector3 object. Vector3 means it is a value with x, y, and z values - i.e position or direction an distance along the X,Y,and Z axis
+ *function update(self, dt)* is called every frame and often this is where we put things that need to be called all the time. (but not the only)
+ *function on_input(self, action_id, action)* this gets called we the app gets input because we got input focus earlier. In this case where we press the keys on the keyboard. We set our speed+direction (velocity - the variable val) here.

We have no code loops in this code. Loops are useful to repeat actions over and over while a certain condition is true. We will get to this, but this book is not a coding book.


---
---

Ok no links here today. Please look for the info I mentioned at the back of the book and the Defold manual. 

PS. When I started using Defold, I bashed away without looking at the core concepts and the addressing and must admit it got *VERY* confusing.
