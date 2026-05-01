layout = "post"
title = "Shoot First, Drive Later"
created = "2026-04-14"
updated = "2026-04-14"
tags = "#reinforcement-learning #artificial-intelligence"
markdown = """
Continuing with my RL experiments, I wanted to try something with physics
this time and also something I can have more fun with using Unreal Engine.

Since I already trained cars in the original Learning Agents plugin, this time
I designed a vehicle that moves like a tank.
I wanted the agent to learn to control left and right tracks independently and learn how to steer without an actual steer action.
I believe this is a good step towards training agents that better understand the physical world.

I put together a box and 6 cylinders for wheels and then added controls for moving the tank with physics. Two keys for left
throttle and two keys for right throttle did the job for human interaction.

<figure>
    <video src="/assets/2026-04-14-shoot-first-drive-later/manual_controls.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>First test of locomotion with keyboard controls</figcaption>
</figure>

Then I wired up the learning part, it got much easier since I switched to C++ because I can structure and reuse the code much better than Blueprints.
It is really satisfying watching the agent having random thoughts and spasms in the environment after wiring up its brain. There is a primal feeling about seeing a virtual being trying to survive in a hostile environment, compared to rule-based AI where you intentionally create its logic.

<figure>
    <video src="/assets/2026-04-14-shoot-first-drive-later/it_is_alive.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>It is alive!</figcaption>
</figure>

As I already mentioned, the actions for navigation are controlling throttle for left and right tracks.
For starters, I decided to reward the agent based on distance delta. If the agent got closer in a step, I gave a positive reward
and if the agent got further in another step I gave a negative reward, both as a function of distance delta.
This worked but contrary to the car example, the tank has no front or back and some of the time 
it learned to travel backwards to the target. To solve this I first decreased backwards speed,
hoping that the policy will figure it out since going forward would yield more rewards.
It didn't work, at least in a short amount of training time. I think once the agent finds a way to reach the target, the direction
of movement is already baked in the network that it is really hard to change later.

I then decided to reward the tank's alignment with the target. Meaning that fully facing the target yields the maximum reward and facing the opposite direction yields the minimum reward, the range being [-1, 1]. This worked nicely, almost.

<figure>
    <video src="/assets/2026-04-14-shoot-first-drive-later/1000.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>Nice agent at 1000 steps</figcaption>
</figure>

The agent learned to navigate quickly to the target just after 1000 steps but as I trained it more to make the movement smoother,
it found an interesting exploit. Because the target moved to a random location when overlapped with the agent, the agent was punished heavily until it faced the target at its new location again. So as soon as inference started the agent was facing the target and moving really slowly towards it, without reaching the target and causing it to relocate, so that it can complete an episode with the highest reward possible.

<figure>
    <video src="/assets/2026-04-14-shoot-first-drive-later/23k.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>Sneaky bastard at 23000 steps</figcaption>
</figure>

To fix this sneaky behaviour, I introduced a reward for actually reaching the target. And it kind of broke my environment. I was assuming that the plugin could only output normalized values, but when I trained the agent with the new reward space(or shape?), the agent became incredibly fast that broke the physics configuration.

Here is the final reward for navigation
<pre class="prettyprint linenums">
FVector TargetLocation = Player->Target->GetActorLocation();
FVector ToTargetPrev = TargetLocation - Player->GetActorPreviousLocation();
FVector ToTarget = TargetLocation - Player->GetActorLocation();

float DistancePrev = ToTargetPrev.Length();
float Distance = ToTarget.Length();

FVector WorldOffset = TargetLocation - Player->GetActorLocation();
// This is what makes the observation egocentric(from the player's perspective)
FVector LocalOffset = Player->GetActorTransform().InverseTransformVector(WorldOffset);
FVector LocalDir = LocalOffset.GetSafeNormal();
float AlignX = LocalDir.X; // 1 if facing directly, -1 if facing the opposite way
float DistanceDelta = DistancePrev - Distance;
Reward += DistanceDelta; // Add distance reward
Reward += AlignX; // Add alignment reward
if (Player->bHasArrived)
{
    Reward += 10.0f;
    Player->bHasArrived = false;
}
</pre>

<figure>
    <video src="/assets/2026-04-14-shoot-first-drive-later/broken_physics.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>Agent breaking the law of physics</figcaption>
</figure>

After debugging some time I realized that the network does not necessarily output normalized values only. The output throttle values were close to 5. That was a big finding for the future experiments as well. I just clamped the network throttle outputs and I was happy with navigation. The vision I had for this agent was a vehicle that can navigate and shoot at the same time. So I moved on to the next lesson.

### Learning Ballistics
I had two options for shooting, hitscan or projectile. I knew that hitscan would be very easy to teach the agent, since it is very similar to the navigation alignment problem. So I decided to go with projectile to make things more interesting.
I added a turret in the back of the tank which proved to be huge pain in the ass, as I'll discuss later in this post. This was also a great introduction to how physics works in Unreal Engine. Coming from Unity, I found that Unreal is not as simple. I especially disliked that labels in the editor and the enum in C++ do not match.

<figure>
    <video src="/assets/2026-04-14-shoot-first-drive-later/caviar_laying_armored_vehicle.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>Behold! The caviar laying armored vehicle, CLAV</figcaption>
</figure>

I decided to start simple. I just let the agent output a random vector for the direction of the projectile, the force factor was a predefined value on the game side. And I defined a reward based on how close the projectile landed to the target. It didn't work at all. As I found out later in the RL book, this kind of reward structure is almost impossible to learn for a PPO network. The projectile was in the air for several seconds and when it landed, the reward was already too disconnected to the action. So it was just random noise for the agent.

To fix this I simply calculated where the projectile might land given the velocity, mass and gravity and rewarded the agent immediately at the time of the shooting action. Since my calculation was simple and didn't take into account uneven terrain and height difference, I needed to level the terrain and also decided to train for shooting without movement. Somewhat a downgrade but it was necessary.

As I mentioned earlier, I added the turret at the back of the tank with a small offset from the center and started training from there. It was a debugging hell. TensorBoard always showed great convergence but whenever I tried inference, it was completely off. Instead of using the slightly offset turret position, the agent was using its own location for calculating trajectory. It found a good approximation which never hit the target but good enough to collect near miss rewards, that's why TensorBoard was looking good. I then calculated everything relative the turret position. Things improved but it was still not great. I then realized that the plugin already did the conversion internally.
I was doing double conversion when writing and reading the direction observation.

<pre class="prettyprint linenums">
ObservationMap.Add("ShellTargetDirection",
ULearningAgentsObservations::MakeDirectionObservation(
    InObservationObject, bShootingEnabled ? WorldDelta : FVector::ForwardVector,
    Player->GetActorTransform())); // Function accepts relative transform
</pre>

After fixing this, shooting has improved a lot but there was still some bias to some orientations. I then realized that
I always spawned the tank with the same position and rotation at every episode. I just randomized those, trained again for some time and voila, the agent was able hit targets with good precision.

<figure>
    <video src="/assets/2026-04-14-shoot-first-drive-later/nice_aim.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>Nice aim!</figcaption>
</figure>

As a final note on shooting, I want to mention that it might seem unintuitive to calculate the actual trajectory just to teach it to the agent, since the work is already done. But there is no way for the network to pick that up on its own. The problem is again the disconnect
between the reward and the action. It is like when training a dog, you reward it with a treat for rolling but the next day. The dog is confused and does not know what the treat is about. Also after studying RL, I realize that humans are not any different.

I had a tank that could drive or shoot and it was time to combine these.

### Combining Driving and Shooting

With these two features successfully converged individually, I then wanted to combine them in a single agent
First I thought, maybe I could make use of multiple interactors but then found that Learnin Agents plugin didn't support that.
Then I thought maybe I can have multiple networks, one for driving and one for shooting. That sounded like a good idea
but it required too much change and I ditched the idea.
As I expected it didn't converge successfully with these two features enabled from the beginning. After 24k steps of training, which is much longer than my usual training amount, the agent learned to drive a little bit but shooting was completely random. On the contrary, both features converged successfully after 2k steps when trained individually.

I already had the curriculum manager from the previous experiment. I cleaned it up and made it more generic and created the simplest curriculum possible.
* First lesson: learn to drive
* Second lesson: learn to shoot

I was hopeful but this didn't work. The agent learned to drive very quickly but couldn't add shooting on top of that.

I then changed the order in the curriculum because I knew that the driving observations and rewards were much more solid than the ones for shooting. And shooting was trained without any movement.
With the new curriculum, the agent successfully learned to shoot first and then learned to drive after around 8k training steps.

<figure>
    <video src="/assets/2026-04-14-shoot-first-drive-later/stop_align_shoot.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>Stop, align, fire, accelerate</figcaption>
</figure>

But again developed a quirky behaviour. Reinforcement learning never ceases to surprise me. When it is time to shoot the agent slows down and aligns, takes a shot and accelerates to the next waypoint. I'm not sure why this happens because the projectile does not have an initial velocity but it might because the shooting training was without movement until driving is enabled by the curriculum.

I was planning to add obstacle avoidance as well but I'll stop here since I'm happy how this experiement turned out. I want to work on
more on physics based locomotion.
"""