layout = "post"
title = "Improvements over Kaboom [DRAFT]"
created = "2026-04-02"
updated = "2026-04-02"
tags = "#reinforcement-learning #artificial-intelligence"
markdown = """
There has beeen a lot of improvements
* Rediscovered curriculum learning and then found out that it was already a well established method in RL. I think it is very natural progression anyways, nothing to brag about. Since RL is modeled after animal learning, that is how you would design experiements with dogs and humans as well. that is what we have in school, start easy and increase difficulty gradually
* Updated the simulation to support agents training in parallel, it sped up training almost 2x
* Increased obstacle observatsion from 1 to 3, that helped a lot but also increased
* Work on proper termination case, for example when the agent hits an obstacle

* Looks like using 599 for no obstacle was wrong, instead I used ArenaHeight and this made no obstacle sentinel to 1.0f exactly, looks promising
* Updated the code so I don't have to start training from scratch every time
* Haing multiple agents running in parallel, in my case 64, is great to avoid a single degenerate stragety

* Fixin Sentinel value improved it a lot. Add sentinel value description, this is very important
* doubled the frequency and continued training at 67k steps

Increased obstacle obervations to 6, use 2.0 for sentinel value

I also seeded the randoms to achieve same type of behaviour every time

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

Architecture Decisions

64 parallel agents provides diverse enough experience that degenerate single strategies are harder to lock in
3 observation slots matched to 3-4 max simultaneous obstacles is well designed — every active threat is visible
Checkpointing every 1000 steps gives you full coverage to recover from policy collapses

Notes on the latest training
Trained on the same parameters for 60k step
Curriculum level 2, Decrease hit count for termination 3->2

POST STARTS HERE
Since the last post, I've played around with the Kaboom example, a lot! I increased obstacle spawn frequency, made obstacles faster and even added collectibles.
All of that worked to a degree but there was something that bothered me a lot.
First of all the agent's movement was jerky. Even though it was smart and sometimes made incredible stunts, other times it just ran into the obstacles or just missed the juicy collectible next to it. 
I wanted to make something that is polished and stable enough to go in a game.
As you can see, the agent from the last post cannot go in a game, it just does not look good.
[Attach the last video from the previous post]

I've spent a lot of time on research and reading. As I kept working on improving the agent I realized that
I needed to improve my workflow otherwise I would lose so much time during trainig. As the agent gets more
complex, the time for training also grows. And there were many times the headless run was corrupted and failed.
I was already saving snapshots, so the natural next step was just being able to load the snapshots and
continue training from there. And then I made the simulation run in parallel. In my setup now I can run 64 agents in parallel. This does not only make training faster but it also provides the network data from 64 different agents simultanously, making the experience diverse enough that degenerate single strategeies are harder to lock in.

These improvements gave me a good starting point. I could now start training, do the dishes, come back and check the results and if anything was not going right, I could return to a good point in the training.

#### Some title
Training an agent to accomplish a certain task and training it to be elegant and graceful are totally different things.
In my simple RL experiments, I was able to train an agent that can avoid obstalce in just 1000 steps.
But when I wanted to make it smooth things went souht really fast.

"""



