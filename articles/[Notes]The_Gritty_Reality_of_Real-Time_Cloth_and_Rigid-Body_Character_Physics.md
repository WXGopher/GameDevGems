## The Gritty Reality of Real-Time Cloth and Rigid-Body Character Physics

adapted from https://www.youtube.com/watch?v=4NkNBImONJU&t



### Chaos cloth 5.0

Difference between Chaos and NvCloth:

![image-20221103205105622](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103205105622.png)

Difference between  backstops (see https://gameworksdocs.nvidia.com/NvCloth/1.1/UserGuide/Index.html#usage for more info)

![image-20221103205753135](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103205753135.png)

![image-20221103210145727](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103210145727.png)

Anim drive stiffness, primarily for blending between simulation and animation.

![image-20221103210250609](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103210250609.png)

This new wind model is kinda cool!

![image-20221103210654512](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103210654512.png)

World collision finally, but this might be costly?

![image-20221103210907438](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103210907438.png)

![image-20221103211241860](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103211241860.png)

Chaos supports proxy mesh.

![image-20221103211345317](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103211345317.png)

And this is how you do it.

![image-20221103211629924](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103211629924.png)

Cloth proxy deformer (mapping between renderable and simulation proxy)

![image-20221103211808950](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103211808950.png)

Some best practices

![image-20221103212519354](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103212519354.png)

Change skeletal mesh workaround

![image-20221103212949329](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103212949329.png)

Important tips:

![image-20221103213355758](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103213355758.png)

cvars

![image-20221103213458245](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103213458245.png)



### Chaos RBAN

RBAN is similar to spring bone(dynamic bone or whatever).

Common problems including:

![image-20221103214246511](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103214246511.png)

Best practices

![image-20221103214302837](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103214302837.png)

![image-20221103214352700](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103214352700.png)

Common techniques

![image-20221103214435864](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103214435864.png)

![image-20221103214625894](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103214625894.png)

![image-20221103214631214](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103214631214.png)

![image-20221103214710994](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103214710994.png)

![image-20221103214746115](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103214746115.png)

![image-20221103214801417](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103214801417.png)

I think the following referenced video is some spring bone stuff, marked for a later visit.

![image-20221103215040583](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103215040583.png)

![image-20221103215220372](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103215220372.png)

Yeah I think fortnite definitely used spring bone...

![image-20221103215341437](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103215341437.png)

Pose driver is important! And we also use this for some pose guiding, like character sitting, etc.

The workflow looks somehow like this

![image-20221103215424898](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103215424898.png)

Combine different parts:

![image-20221103221351522](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103221351522.png)

### Chaos 5.1

![image-20221103221928380](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103221928380.png)

![image-20221103222133028](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103222133028.png)

![image-20221103222145223](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103222145223.png)

Note: self collision is expenseive!

![image-20221103222213904](.\[Notes]The_Gritty_Reality_of_Real-Time_Cloth_and_Rigid-Body_Character_Physics.assets\image-20221103222213904.png)



### Learning resources

https://dev.epicgames.com/community/learning/paths/QX/unreal-engine-welcome-to-chaos-character-physics

