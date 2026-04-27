layout = "post"
title = "Stepping into Physics [DRAFT]"
created = "2026-04-14"
updated = "2026-04-14"
tags = "#reinforcement-learning #artificial-intelligence"
markdown = """
[DRAFT]
Continuing with my RL experiments, I wanted to try something with physics
this time and also something I can have more fun with in Unreal Engine.

Since I already trained cars in the original Learning Agents plugin, this time
I designed a vehicle that moves like a tank.
I wanted the agent to learn to control left and right tracks independently and learn how to steer without an actual steer action.

// This is actually a good start, I now realize that steering without a steering action is very interesting.

// you can add the video that you control the tank manually

I put together a box and 6 cylinders and added controls for moving the tank with physics.

<figure>
    <video src="/assets/2026-04-14-stepping-into-physics/manual_controls.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>First test of locomotion with keyboard controls</figcaption>
</figure>

Then I wired up the learning part, it got much easier since I switched to C++ because I can structure and reuse the code much better than Blueprints.
It is really satisfying watching the agent having random thoughts and spasms in the environment after wiring up its brain. There is a primal feeling about seeing a virtual being trying to survive in a hostile environment, compared to rule-based AI where you intentionally create its logic.

<figure>
    <video src="/assets/2026-04-14-stepping-into-physics/it_is_alive.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>It is alive!</figcaption>
</figure>

As I already mentioned, the actions for navigation are controlling left and right track throttle.
And I decided to reward the agent based on distance delta. If the agent got closer in a step, give a positive reward relative to the distance delta and if the agent got further in another step
give a negative reward, again relative to the distance delta.
This worked but contrary to the car example, the tank has no front or back and most of the time 
it learned to travel backwards to the target. To solve this I first decreased backwards speed,
hoping that the policy will figure it out since going forwards would yield more rewards.
It didn't work, at least in a short amount of training time.

I then decided to reward tank's alignment with the target. Facing the target yields more positive and facing the opposite direction yields more negative rewards. And I kept the distance reward. This worked nicely, almost.

The agent learned to navigate quickly to the target just after 1000 steps but as I trained it more
it found an interesting exploit. Because the target moved to a random location when overlapped with the agent, the agent was punished heavily when it reached the target. The exploit was facing the target and moving really slowly towards it so that it can complete an episode with the highest reward possible.

// You might want to speed up or combine the videos
<figure>
    <video src="/assets/2026-04-14-stepping-into-physics/1000.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>Nice agent at 1000 steps</figcaption>
</figure>

<figure>
    <video src="/assets/2026-04-14-stepping-into-physics/23k.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>Sneaky bastard at 23000 steps</figcaption>
</figure>

Interesting findings
* It was relatively easy to teach the agent to follow a target, just by rewarding it for moving closer to the target and facing the target
float DistanceDelta = pd - d;
// AlignX is between [-1, 1]
OutReward = DistanceDelta + AlignX;

To fix this sneaky behaviour, I introduced a reward for actually reaching the target. And it kind of broke my environment. I was assuming that the network could only output normalized values, meaning values in [-1, 1] range but when I trained the agent with the new reward space(or shape?), the agent became incredibly fast that broke the physics configuration.

<figure>
    <video src="/assets/2026-04-14-stepping-into-physics/broken_physics.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>Agent breaking the law of physics</figcaption>
</figure>


After nailing projectile shooting, you can compare its performance against heuristic projectile shooting.


Adding multiple interactors, which enables toggling features much easily. Later: You can only have one interactor per (Policy, decoder, encoder, cricit) group.

summary from claude chat 1
Egocentric Observations
Using InverseTransformVector to convert world-space offset into actor-local space, giving the agent rotation-invariant observations. AlignX and AlignY are the normalized direction components to the target from the agent's perspective.
Reward Shaping
Final reward formula: DistanceDelta + AlignX - 0.01f. The AlignX term naturally discourages the agent from reversing toward the target. The tick penalty discourages idling. A completion bonus of +10.0f on overlap was critical — without it the agent learned to approach but never commit to reaching the target.
Lazy Policy Problem
Without the completion bonus the agent converged on a local optimum: face the target for AlignX reward, don't touch it to avoid the repositioning penalty spike. Visible on TensorBoard as flat episode length at 500 and ~1.0 avg reward.
Unbounded Action Space
SpecifyFloatAction scale parameter is a hint to the network, not a hard clamp. The agent learned to output values up to 4.0 despite a 1.0f scale definition, moving 4x faster than intended and eventually breaking the physics. Fix is to clamp in SetThrottle. This is undocumented behavior in Learning Agents worth calling out explicitly.
Emergent Behavior
The agent naturally developed a two-phase approach: drive toward target at distance, align precisely when close. This wasn't designed — it fell out of the reward function.

IDEA: I had this idea of implementing a small neural net in rust, like 5 nodes, 1 hidden layer. To simulate a small part of this learning experiement but I forgot. Maybe it was a network that learned to aim.
"""