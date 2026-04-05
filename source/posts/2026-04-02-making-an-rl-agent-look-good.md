layout = "post"
title = "Making an RL Agent Look Good [DRAFT]"
created = "2026-04-02"
updated = "2026-04-02"
tags = "#reinforcement-learning #artificial-intelligence"
markdown = """
Observation Space

Sentinel values for empty obstacle slots must normalize to an unambiguous value — 2.0f is cleaner than 1.0f which is indistinguishable from a real near obstacle. This single fix produced the most dramatic training improvement of the day
Separate dx/dy is correct for box colliders — Euclidean distance would give the network a false radially symmetric threat model
Obstacle slots should be fixed at the maximum you'll ever need from the start, using sentinels for empty slots — changing architecture mid-curriculum breaks warm starts
Sort obstacles by Euclidean distance not vertical distance — vertical sort misroutes attention when horizontal threats are more imminent

Reward Shaping

Episode termination on hit is already a very strong signal — a large hit penalty on top of it may be redundant and can cause overly passive behavior
Pure negative rewards can produce degenerate policies where dying quickly is optimal — a small survival reward per tick counteracts this
Simpler reward functions are generally easier for PPO to optimize cleanly
Rewarding obstacle hitting the ground creates a confusing association between "obstacle very close on Z" and positive reward

Curriculum Learning

Gate phase transitions on actual performance — avg episode length approaching the cap — rather than fixed step thresholds
Increase difficulty incrementally (25-30% per phase) rather than large jumps like doubling
Keep architecture fixed across phases and only change environment parameters — this preserves warm start capability
Matched observation slots to obstacle density is critical — asking the agent to dodge invisible threats is not a learnable problem

Training Dynamics

Smoothed TensorBoard curves hide important variance — always zoom into raw curves to see true stability
Sharp drops followed by recovery indicate learning rate is too aggressive for the current training stage — the policy is bouncing around the optimum rather than settling
Plateaus followed by resumed climbing are normal PPO behavior — the policy consolidates before finding the next gradient
Hitting the episode length cap repeatedly is a clean signal the agent has mastered current difficulty
True reproducibility in parallel RL training is extremely hard — seeds give approximate reproducibility but thread scheduling and float accumulation cause divergence

POST STARTS HERE

Since the last post, I've played around with the Kaboom example, a lot! I increased obstacle spawn frequency, made obstacles faster and even added collectibles.
All of that worked to a degree but there was something that bothered me a lot.
First of all the agent's movement was jerky. Even though it was smart and sometimes made incredible stunts, other times it just ran into the obstacles or just missed the juicy collectible next to it. 
I wanted to make something that is polished and stable enough to go in a game.
As you can see, the agent from the last post cannot go in a game, it just does not look good.

<figure>
    <video src="/assets/2026-03-14-teaching-an-ai-to-play-kaboom/reversekaboom_improved.mp4" controls playsinline>
        Your browser does not support the video tag.
    </video>
    <figcaption>Nervous agent</figcaption>
</figure>

I've spent a lot of time on research and reading. As I kept working on improving the agent I realized that
I needed to improve my workflow otherwise I would lose so much time during trainig. As the agent gets more
complex, the time for training also grows. And there were many times the headless run was corrupted and failed.
I was already saving snapshots, so the natural next step was just being able to load the snapshots and
continue training from there. And then I made the simulation run in parallel. In my setup now I can run 64 agents in parallel. Only this cut the training time in half. This does not only make training faster but it also provides the network data from 64 different agents simultanously, making the experience diverse enough that degenerate single strategeies are harder to lock in.
I also made sure that the random generators worked the same every run because I noticed there were significant difference when the same network run multiples times.

These improvements gave me a good starting point. I could now start training, do the dishes, come back and check the results and if anything was not going right, I could return to a good point in the training.

#### Some title
Training an agent to accomplish a certain task and training it to be elegant and graceful are totally different things.
In my simple RL experiments, I was able to train an agent that can avoid obstalce in just 1000 steps.
But when I wanted to make it smooth things went souht really fast.
I first started with improving the observation space. I increased observed obstacle count from 1 to 3 and later to 6.
To observe multiple obstacles I sorted them by the their euclidian distances to the agent.
Having more observations definitely helps but it also increases training time.
I also added a termination rule which didn't exist before. If the agent got hit 3 times, it was out.
But the real breakthrough came after I fixed the sentinel values. Neural networks work best with normalized values, instead of working with absolute distances. In my simulation the agent is always at z=0.0 and obstacles spawn at z=600.0. Instead of passing the distance in absolute UE units, I normalize them. 

normalized vertical distance = absolute vertical distance / absolute max distance
since distance decreases as the obstacle falls, a trick is reverting it with 
normalized = (1 - normalized)
this way when the obstacle spawns the distance is 0 and when it is about the touch the ground it is 1.

But this meant if there is no obstacle it is also 0, this is the sentinel value. Sentinel value is the similar to null for observation value. when ther is no obstacle basically. So instead of 0, I just passed 2. 2 is arbitrary here, the important thing is having a value outside normalized range.
In the image below you can see a total of 60k training steps.

<figure>
    <img src="/assets/2026-04-02-making-an-rl-agent-look-good/first_three_runs.png" alt="Different observation spaces">
    <figcaption>TensorBoard graphs showing three consecutive runs of the same network</figcaption>
</figure>

* Orange: From 0 to 20k, the agent slowly learned to avoid obstacles
* Blue: From 20k to 41k, the agent was still improved but not much, you can see that episode lenght is at 500
* Red: From 41k to 60k, I think this was not necessary since the agent stayed the same

This was a good run but there was more to improve. I was not really sure about the miss reward, the 1.0 reward I gave every time an obstacle touched the ground, meaning the agent successfully avoided it. I removed it and the agent did not care. Because the -10.0 reward for getting hit is more than enough to avoid obstacles. Miss reward was just noise for the network.

<figure>
    <img src="/assets/2026-04-02-making-an-rl-agent-look-good/remove_miss_reward.png" alt="Different observation spaces">
    <figcaption>Pink curve does not have the miss reward, trained on top of orange</figcaption>
</figure>

While cycling to the rowing club and simultanously thinking about RL, I realized I might improve agent behaviour by introducing difficulty gradually. Not to brag but I basically rediscovered a concept called Curriculum learning, which is an established method in RL.
Honestly there is nothing to brag since this is only natural progression.

- you can add a proper definition of curriculum learning

these are for later
* Cyan: Decreased termination rule from 3 to 2, agent struggled a lot but inference was not that bad
* Magenta: Here I removed miss reward, and continued from 60k steps, the agent was completely fine. Even though average return fell down, episode length was a solid 500. inference was the same, this tells that miss reward was just noise for the agent
* Green: I changed -0.01 reward every tick to 0.02 reward every tick, this is also after 60k steps, the agent collapsed
* Gray: same as green from 0, it didn't converge at all
* Orange 2: increased obstacle frequence(2x) continued from pink

"""



