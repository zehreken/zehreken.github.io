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
"""