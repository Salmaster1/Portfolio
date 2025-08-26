[<< Back](https://salmaster1.github.io/Portfolio/)

# Terminal Velocity  

Terminal Velocity was developed over the couse of two-three -ish weeks as a part of me learning Unreal Engine during my time at Yrgo.  

Terminal Velocity was made using Unreal Engine's visual scripting language called "Blueprints", and this was my first time using this language.  

Terminal velocity is a racing game where you are a ball that can throw orbs that can either attract you to them or repel you from them. Using these you have to pass through all checkpoints and eventually reach the goal.  

## Movement

The movement is pretty simple. WASD to move, Space to jump. Thats it.  

There are some more factors that help make the movement feel better. The movement is acceleration-based, and in order to make is easier to turn around, there is a special section of the code that increases your acceleration when accelerating in a direction different from your velocity.  
What this does is practice is make it easier to change direction when moving.  

Towards the bottom of the screen, there is a text field that displays the player's horizontal speed.  

The player's velocity is affected by friction, and when the player surpasses a speed of 4000, a secondary speed-reduction system activates, which proportionally reduces the player's speed the higher over 4000 they are. As such, on the ground, the player's max speed is around 5500-6000.  

While in the air, the player has no input, and has (basically) no friction. As such, a well-placed superjump at high velocities can be very rewarding, at the risk of flying way off-target.  
In adittion, while airborde, there is no soft-cap of 4000 speed, allowing the player to go **much** faster. With the right setup and technique, you can reach an aerial speed of ~20000, sometimes way higher.

## The Gravity Balls

The following is a screenshot from the game:  

![Screenshot woo hoo!](/assets/TerminalVelocitySS.png)

In the bottom-right corner, there is a change-bar and well as two prompts, RMB and LMB.  

When pressing either button, a gravity ball will be shot towards the in-game crosshair. The ball's polarity is determined by what button shot it: Attract for LMB and Repel for RMB.  

The strength of the ball's gravity is determined by the change-bar. By using the scrollwheel, you can change the strength of balls you shoot. Balls shot at 100% power will have very strong gravity over a large area, while a 0% ball will have *extremely* weak gravity over a small area.  

The ball's velocity through the air is calculated in a way that should in most cases prevent you, the player, from outpacing it, and it should as such always land in front of you (assuming that you shot it forwards).  

The balls themselves are not affected by gravity, neither their own nor the global gravity from Unreal Engine itself. They always fly in a straight line.  