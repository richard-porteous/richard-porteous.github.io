## Hero RPG a Role Playing Game

---
Defold tutorial has been put on hold. I will get back to it but I need to get past the lack of useful documentation they provide. Untill then I'll be writing on other Game engines and other topics. This has been left in the blog simply as a reference.
---


### What we will do this Lesson:
+	Design and Layout how the game scene will look.
+	Create or download some art work.
+	Build the first scene.

### Let's design the Game

When designing a game it is always a good idea to look at a basic idea for the game (ok there is much more than that - but this is a beginners book, so I kept it short).


### Story:

Lets create an essential experience story and then grab the Key-words.

+ We are on the last day of a vaction in a town where the people are suddenly afraid to leave and have barricaded the town. The mayor is hiding something and if we want to leave the town we have to find out what is going on. We are about to become a reluctant hero.

We could do better, but the key-words are: 

### *Essential Experience*:

+ secretive (mayor is hiding something)
+ urgency (we want to leave the town)
+ dangerous world (reluctant hero)
+ RPG (role-playing-game)


### *Game play factors*:

+ Walk and run (movement keys plus shift key)
+ Conversation dialogue
+ ~~pick up items~~
+ ~~change to other scenes and back.~~

In this partial RPG game we want the player to guide the Hero around the town looking for answers. We need to talk with people and go find answers ourselves when no one wants to answer us. We may have to visit other areas of the town without being seen and sometimes we may need to pick up things to help us complete our goal.

I'm using this as an example. We focus on making a town, a player, an NPC, and show a message from the Mayor when the player walks up to the Mayor. The NPC doesn't move in this part of the game to keep it simple.

### Create Project


**NOTE - SAVE OFTEN** using *Ctrl-S*


If you haven't already created an empty project lets run godot an do this - we will name it HeroRPG and create the project.

![](/img/session1-images/create_project.png)

This will open the project. If you have closed Defold since you created the project you can open it from this screen. Look for the one you just created and open it.

![](/img/session1-images/opening_existing_project.png)


Then we adjust the screen size in our project settings. The file is *game.project*. look on the left you can double click the file. Scroll down till you find the Display settings and can change the size like so. This is an old common size used for the iPad. You can skip this, but notice that all the settings you need to cange to match the device you want to target should be found here.

![](/img/session1-images/adjust_the_screensize.png)

### Add Assets

You can download these from ["assets"](https://drive.google.com/drive/folders/15ZRZfJun1KA3JR7RznVJj3ge_Pn8TzcW?usp=sharing). You will to extract them.

Let's Add the art to defold and get prepared to start actually building the game. We are going to use place holder art work i.e. nothing fancy preferably free.

So I'm not an artist. I simply need something for this book. I used paint3d (it has transperancy and 2D shapes) and I made the buildings. I used Gimp to fix the one.

![](/img/session1-images/save_a_few_assets.png)

You will notice I went and downloaded Bevouliin Free Game Background for Game Developers from Open-Game-Art (Page Links at the end of the session).

### Import the Background and Town Scene Images.

Right click the top level folder. It will match the name you used i.e. HeroRPG. A drop down menu appears and select *new folder*. Name it *assets*. 

This is where we will place all our unmodified png and jpg files. When naming any file use lower-case and no spaces. Use an *_* underscore to seperate words if you want. This is standard practice for file names as it avoids OS problems with *Linux, Mac, Android* and *ios*.

### When you get images

Drag the downloaded images (png or jpg) into the assets folder. If you dont have an assets folder create one now.

![](/img/session1-images/assets_folder.png)


### BUILD THE FIRST SCENE

As promised lets build the first scene.

I did metion where you can get artwork for free to build your prototypes. But for simplicity copy the extracted files and folders into your newly created assets folder. You will have enough to build the scene.

![](/img/session1-images/open_main_collection.png)

Single click on Main and double-click on the main collection to open it. Notice on the right window we see in the *Outline* the Collection. (function key f8 toggles the outline view).
We need to add our background, buildings etc here.

I will spend a few minutes explaining what is happening in the next session, for now lets merrily go on our way and build the scene.

first we need to create an atlas so we can use the images. Right-click the main folder and select new atlas.

![](/img/session1-images/create_an_atlas.png)

We are only using one atlas for this scene so lets just call it assets.

![](/img/session1-images/name_atlas.png)

on the right of the editor (with the assets.atlas selected) right-click Atlas and select *Add Images*

![](/img/session1-images/add_images_to_atlas.png)

Select All the background images and click ok

![](/img/session1-images/add_background_images_to_atlas.png)

We now have serveral individual images to use for our scene.

![](/img/session1-images/our_atlas_with_background.png)

Lets reselect our main selection. It still is open so you can simply select its tab.

![](/img/session1-images/reselect_the_main_collection.png)

Right-click the collection on the right and select *Add game Object*. In the **Id** box rename it *background*.

![](/img/session1-images/first_game_object.png)

For now always use lower case names and use *underscore_to_seperate_words* known as *snake case*. If the editor creates things using the first character as uppercase its ok.

Right-click the background game-object and create a sprite as a child component.

![](/img/session1-images/add_a_sprite.png)

Add the atlas as the sprite image.

![](/img/session1-images/add_atlas_to_sprite.png)

Select the layer_1 image as Default Animation.

![](/img/session1-images/select_the_image_for_the_sprite.png)


*Taken directly from the editor manual*

The Editor pane

    The center view shows the currently open file in an editor for that file type. All visual editors allows you to change the camera view:

    Pan: Alt + left mouse button.
    Zoom: Alt + Right button (three button mouse) or Ctrl + Mouse button (one button). If your mouse has a scroll wheel, it can be used to zoom.
    Rotate in 3D: Ctrl + left mouse button.

A first look at our new background. You likely have to **zoom** out. Click on the main view and use the scroll wheel to zoom in or out. It may start too close so zoom out to get this view.


![](/img/session1-images/background_sprite.png)

Using the order or furthest to closest we can add the various sprites as we go.

+ Click Collection.
+ Create another gameobject called street.
+ Click Collection again.
+ Create another gameobject called buildings.

Add the sprites - you can name them to make it easier to know which sprites they are and because you cant have the same name for each component in an object.

![](/img/session1-images/add_remaining_background_objects.png)

### OK WHY CANT I SEE THE OTHER SPRITES?

We need to change the z position of the game object. Its how we can see the other sprites.

Lets move the background back. Change its Z in Position to -1.

![](/img/session1-images/change_background_z.png)

Lets change buildings to -0.6
and street to -0.7

Now select buildings and grab its vertical (green) arrow and drag it down, and select each building sprite. move hospital left using the red arrow (horizontal) and move the store right, leaving the library in the middle.

![](/img/session1-images/move_buildings.png)

Ok Zoom in, and recenter. See editor section for navigating in a scene or use the middle mouse scroll wheel as a button. 

We put the street slightly behind the buildings as we want thier fronts to appear on the sidewalk. Lets move the sidewalk and the pavement down. Movethe group then move the individual sprites to get them positioned correctly. a sligth overlap of the sidewalk over the pavement. set pavement z -0.1. 

![](/img/session1-images/move_street_down.png)

and scale the street 2x them to cover half the scene.

![](/img/session1-images/scale_street.png)

Now we need to duplicate and flip it on the Y rotation and move it to the right side. We flip it because of the lines on the sidewalk give an idea of perspective and it would be odd not to keep that in mind.

Select both sprites of the street object. Use Ctrl-C and Ctrl-V (copy & paste) to duplicate them. with the copies selected drag them both over to the right until they ony slightly overlap. rename them, by changing thier y rotation to 180.

![](/img/session1-images/street_sprites_copied_and_rotated.png)


We have a scene. Its very basic, but good for our needs.

I'll repeat this - just read through it now.
### ***Core Concepts*** (brief)

+ Collection.
    + A collection is a file used to structure your game. It holds **game-objects and other collections**. They are typically used to structure game levels, groups of enemies or characters built out of several game objects.
    Our main scene was a collection.
+ Game object
    + A game object is a container with an id, position, rotation and scale. It can hold other **collections**, **game-objects**, and **components**. They are mostly used to create the "things" in a game.
    The player, NPC, buldings, street,  sidewalk and background were game-objects.  
+ Component
    + Components are entities that are put in game object to give them **visual, audible and/or logic representation** in the game (see, hear, do). They are typically used to create character sprites, script files, add sound effects or add particle effects. The sprites we used are components.

Read core concepts in the manual for more detail, now is a good time to do it.


Next session we add the Player and a NPC character.

---

#### **SITES**

Here is a list of sites for you to look at, for your artwork.

+ Kenney.nl - used by many in tutorials. Many of these have been made specifically to run in Godot, but they'll do well for our needs.

+ craftpix.net - This one is like an asset store.
+ tokegameart.net - so is this one
+ gamedevmarket.net - and this 

+ OpenGameArt.org - been around a long time
+ GlitchtheGame.com - game that was never released
+ Gameart2d.com
+ Shutterstock.com

please - sites come and go, don't trust them if they look odd to you.

#### Fonts
+ dafont.com  - make the text in your game look just right.

Some sites listed may require the artwork to be converted to use with Godot (wrong format).

Always check licences of other peoples work. Be aware of the restrictions.




### **Page Links**

+ Code repository for the book https://github.com/richard-porteous/MakeGamesWithDefold_projects

+ Assets for this series of books https://drive.google.com/drive/folders/15ZRZfJun1KA3JR7RznVJj3ge_Pn8TzcW?usp=sharing

+ Bevouliin Free Game Background for Game Developers from Open-Game-Art https://opengameart.org/content/bevouliin-free-game-background-for-game-developers

Defold Editor - https://defold.com/manuals/editor/
Importing Images - https://defold.com/manuals/importing-graphics/ and https://defold.com/manuals/importing-assets/


