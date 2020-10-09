# AI Final Project Spring 2020 - Jacob Stallings
## Introduction
The following a study of the famous Pacman problem, the conundrum that the classic game presents of collecting all the dots in the maze while avoiding ghosts. Though for our purposes, we disregarded the spooky ghosts, the challenge still stands to develop the ultimate Pacman playing algorithm. I present a description of the challenge by first describing the way the search algorithms are set up and implemented in the code, then comparing different algorithms for the two problems we'll face: finding the shortest path to one particular spot in the maze, and finding the shortest path through the maze that lets us eat all the food dots (the classic Pacman problem). The comparison pits algorithms against each other for mazes of increasing complexity and size to find out which ones work the best for different cases. Finally, I draw some conclusions from the data of runtime, nodes expanded, and path cost found by the different algorithms to explain why the different algorithms work the way that they do.
## Description
### Understanding search algorithms
#### Why do we have dummy methods in `search.py`? ###
`search.py` contains the abstract class `SearchProblem`. The abstract class allows us to define an abstract problem structure that we can then make concrete according to the type of search problem we are being asked to solve. For example, if the problem is to find a path to the one piece of food in the game (one-food problem), we define different methods of getting our successor states, than in the other type of problem, where we are trying to find the path that lets us eat all the food. Different search problems can differ in attributes such as start state, goal test, successor, function and cost function, and the abstractness of `SearchProblem` allows us to adapt those attributes according to our needs, which means that we can use the abstract methods when defining our search algorithms - regardless of the structure of the problem, the search algorithms, which are well defined, always need to do things such as get their successors, check the goal state, etc., but depending on the problem the actual succesors or goal state themselves will be different.

#### What's the meaning of the `structure` parameter?
The `structure` parameter in the function `generalGraphSearch()` in `search.py` refers to the data structure to be used to keep track of states that we have already visited along our path. As we know, a searching algorithm is capable of producing a list of states, list of actions (policy) or cost that gets us to our goal in the most optimal way. This means that any searching algorithm must keep track of the actions that it has taken for *every* path in case that path ends up reaching all the way to the goal. The different well-known searching algorithms are similar in concept but differ in the way in which successor states are explored. For example, DFS uses a LIFO (Last in, first out) scheme to discover new unvisited nodes, so we use a stack, which has a LIFO structure, whereas BFS uses a FIFO (First in, first out) scheme to add unvisited nodes to the list of nodes we need to visit, so we use a queue, the FIFO data structure, to store this list. In general, when the searching algorithm discovers an undiscovered state, it adds it to the data structure storing the states to expand, then pops off the data structure when it is time to expand that node. Depending on the data structure, this process can be different, but the concept of pushing and popping the data structure remains the same, therefore we can write the searching algorithm for an arbitrary data structure and then take in which data stucture we want to use as a parameter which also determines what kind of search we perform.
Here is a summary of the graph search methods used in this project and the data structures that each uses:
- **DFS**: Uses a stack, successors are pushed to the stack then popped off and explored instantly
- **BFS**: Uses a queue, unexplored successors are pushed but not popped until it is time to explore their level (i.e. the sucessors 5 steps away from the start will not be explored until the successors at levels 1, 2, 3, and 4, which are ahead of them in the queue, are popped off and explored)
- **UCS / Dijkstra**: Uses a priority queue, where the sorting function is f(x), the cost function
- **A***: Also uses a priority queue, but the sorting function if f(x) + h(x) , where h(x) is the chosen heuristic function

#### How do we implement most part of the search algorithms in a single method (generalGraphSearch)?
As described above, the different search methods such as DFS, BFS, Dijkstra, and A* are all similar to each other - they are all Graph searches, but each is slightly different. Since the general format of each algorithm is essentially the same (add unknown nodes to a data structure in a certain way, then pop off each node and return the optimal path), it makes sense to write one general procedure for a general graph search, then use different, more specified versions of that procedure depending on which search we do. This means that all we have to do for each specific implementation for each algorithm is simply specifiy the data structure to be used and then pass that as a parameter to our generalGraphSearch function.

We have our generic search algorithm `generalGraphSearch`. We also have our abstract class `SearchProblem`. These two general elements can be modified to choose the algorithm and problem type. There are 4 algorithms & 2 problem types -> 2*4 = 8 possible combinations. That's 8 different methods that have been combined into just two! Following is an analysis of both problem types and how each is connected to a search algorithm.
### Problem 1: Finding a fixed food dot
For this problem, we want to find the least cost path (shortest path) to a food dot in a fixed position on the grid.
- State space: (x,y) positions in a Pacman game.
- Initial state: Start position of the Pacman agent as indicated by placing a P at the desired start position in the layout file of each maze.
- Goal state: The position of the fixed food dot. In this study, it is always the bottom left corner regardless of whether there is an actual dot there.
- Operators: Move N, S, E, or W
  - For each operator, if the successor state (position) as a result of applying that operator would be a wall, the operator cannot be applied and we remain in the current state.
  - For each operator, the result of its application is as follows:
    - For N, s' = (x,y+1)
    - For S, s' = (x,y-1)
    - For E, s' = (x+1,y)
    - For W, s' = (x-1,y)

#### Analysis
I designed four different mazes of incremented complexity to test the behavior of our search algorithms. The smallest is userSimple.lay, followed by userTunnel.lay, userSpirals.lay, and finally userHuge.lay . The more complex mazes feature spiral structures which I put in to see to what degree each search algorithm follows them. Some of the spirals have dots, some do not and would be pointless to go down. (In the case of this problem, it's never worth sacrificing distance for food, as we don't care how much food we pick up. But if we wanted to test the second type of problem with these grids we might get more interesting behaviour with regards to the sprials.I'm interested to see the behavior specifically of the DFS on these parts.)  
The first time I collected data, I found after running the algorithms that the optimal solution in all of my mazes was to just travel more or less straight down the left side because the goal is always in the bottom left corner. I then changed the layouts so that the Pacman would have to venture farther over towards the right side of the maze. Sure enough, I was able to see a lot more separation between the runtimes for the different algorithms.

Below are the charted data for the four different algorithms run on the four different puzzles.

![Execution Time (ms) vs. Layout](./img/ExecutionTime.png)
*Figure 1*
  
![Expanded Nodes vs. Layout](./img/ExpandedNodes.png)
*Figure 2*
  
![Path Cost vs. Layout](./img/PathCost.png)
*Figure 3*  
Note: Path Cost for A\*(M), A\*(E), and BFS are all the same


After examining the data, here are a few conclusions:
##### BFS expands the most nodes
It's clear that BFS expands more nodes on average for a given maze, and this makes sense because BFS may examine nodes that are nowhere near the goal, but are the same distance away. For example, if we are 20 away from the goal, but also 20 away from the bottom right corner, there's an equal chance that BFS explores the bottom right corner before the bottom left, because they would both be at level 20. Because BFS tends to "dip its toes" in all possible paths rather than following deeply down a potential solution, we expect it to expand more nodes where many of those nodes are not part of a potential solution.
##### DFS is quick, but rarely finds the optimal path
Surprisingly, BFS expanded more nodes but always found the optimal path, whereas DFS only found that optimal path in one out of four cases (userTunnel). This is most likely due to the fact that DFS is naive and does not have a heuristic to judge if the path it is following down is actually the most optimal solution. Because DFS will go deeper and deeper then return once it finds ANY solution, there is no guarantee that the first solution it finds is the optimal one. The reason BFS works so well in contrast is because distance from our start point plays a role in what nodes we explore next. Therefore, we always find the solution that is the shortest distance from our start, but the tradeoff is that it BFS longer because we have to explore all those extra nodes that are closer to us.

##### A* is solid, Manhattan Distance is king
Rather unsurprisingly, the heuristic searches always found the optimal path, and did so in a short enough amount of time. The Manhattan distance proved to be the better heuristic funciton, as A* Manhattan was the clear winner in both minimal execution time and minimal nodes explored for larger puzzles, namely the userHuge one (for the smaller puzzles, the difference was more or less negligible).
If you look at the userHuge puzzle, the difference between the two heuristic searches really becomes stark. *Why is A\* so much quicker and more efficient when using the Manhattan distance as opposed to Euclidean?* The answer lies in the way the distances are calculated as well as the fact that Pacman can't move diagonally. Take the example of an empty grid. When we use Euclidean distance, 5 units above the goal is the same thing as 5 units diagnoal to the goal, but the diagonal will take us twice as many moves to traverse because each node that is diagonal to us requires us two moves to explore it (one vertical and one horizontal.) Because the Manhattan distance reflects accurately the fact that it is more costly to explore a node that is diagnoal from the goal than to just keep going horizontally or vertically, even though it might be slightly farther away in Euclidean terms. Because Pacman cannot move diagnoal, the Manhattan distance is a slightly more accurate representation of our grid environment - the total number of moves up, down, left, and right is what we really care about. For example, in userHuge, the biggest maze tested, when we have few walls obstructing us towards the end, Euclidean explores a lot more open space than is necessary. Manhattan simply goes all the way down, then all the way left. This is more time-efficient. (Note that the Euclidean version still finds the optimal path reliably, just takes slightly longer). This goes to show that a well-chosen heuristic that models your problem accurately can have a great impact on algorithm performance.  

##### Case where DFS beats A\*(M)
The file `userDFSLess.py` contains the maze designed so that DFS expands less nodes than A* Manhattan while still finding the optimal path. In this case, DFS expands 51 nodes (the length of the optimal path) while A* Manhattan expands 142 nodes.  
The maze design is simply a modification of the `userSpirals` design with a wall on the right side keeping Pacman from being able to explore the center of the map once he has started down the right side. This design works because there is a wall in the bottom left corner, allowing Pacman to explore up to that point, but then be unable to reach the goal. In this case, *shorter distance to the goal does not actually indicate a better option*, since the optimal path is to go right from the start and take the path that goes along the border of the map on the right hand side, as this is the only way to reach the goal. Manhattan distance is misleading because we actually have to go farther away from the goal to be closer to actually reaching it. Because DFS starts down the correct path initially, it never even has to backtrack.  

##### Manhattan A* never expands more nodes than BFS
Because of the way the two algorithms are designed, A* with a Manhattan will never expand *more* nodes than BFS. This is because the A* algorithm expands the subset of nodes that BFS expands which are closer to the goal rather than farther away. They both use a queue, so in many ways A* behaves similarly to BFS, but the real crucial part comes when we are really close to the solution, but there is a fork in the road. Because A* uses a PRIORITY QUEUE, rather than a normal queue, it will always explore the nodes that are closer to the goal first at this point, and usually that means we will find the goal, and won't need to explore that other fork. Running the two algorithms on the userSpiral maze file previously described yields a situation where BFS searches just two more nodes than A* because there is a fork right next to the goal that BFS must explore even though it takes it farther away from the goal, whereas A* will never explore it because that fork's nodes are farther away in the priority queue and never get expanded. Figures 4 and 5 below shows this result. The first is BFS, and the second is A* Manhattan. As we can see, the two extra nodes come from the way BFS is built to explore equally in all directions while A* Manhattan has a preference for going down and left (towards the goal).  
![BFS solving the userDFSLess maze](img/userDFSLessBFS.png)  
*Figure 4*  

![A* with Manhattan heuristic solving the userDFSLess maze](img/userDFSLessAStar.png)  
*Figure 5*  

The best we can do, then, is design a maze where the two algorithms expand the *same amount* of nodes. This was done by blocking off the fork in the bottom left corner so there is nothing for BFS to explore there. The result is the file `userBFSEqual.lay`.  

### Problem 2: Eating all the food dots
- State space: Tuples of the form (pacmanPosition, foodGrid) where position is a tuple of the form (x,y) corresponding to the xy-position of the Pacman, and foodGrid is a Grid of boolean values specifying the remaining food: true if the position contains a food dot, false otherwise. *width* x positions, *height* y positions and each with width\*height\* possible values for foodGrid gives a total number of (width^2)*(height^2) possible states.
- Initial state: A tuple of the start position denoted by the positioning of the letter P in the layout file, as well as an initialized foodGrid where the value is true wherever there is a food dot initially and false everywhere else.
- Goal state: Any state tuple where the foodGrid contains all false values - the x,y position doesn't matter.  

#### Description of Agents
##### AGENT1: ClosestFoodManhattanDistance
This agent uses A* to find its way through the maze with the heuristic being the Manhattan distance between the Pacman and the farthest food dot. In this case the estimated cost to the goal is the cost to get to the food that is farthest away from our Pacman's position.  
##### AGENT2: ClosestFoodMazeDistance
This agent also uses A* to find its way through the maze, but the difference between this agent and the first one is the heuristic function used: for this heuristic we calculate the exact number of moves we would need to reach the farthest food dot. This means we have to account for walls when calculating, whereas we don't when we use the Manhattan distance. For this reason, the maze distance seems like it should be the best heuristic function to use.  
##### AGENT3: ClosestDotSearch
This agent is slightly different from the other two, it always searches for the food dot that is closest to it and goes directly to it, then from there searches again to find the closest dot from there, and repeats until all the dots are gone. The search performed is a BFS using the current state of the Pacman as the starting state, the search returns a list of actions for the pacman to take to the closest food dot.

#### Plots
Below are the plots for certain mazes for each agent. Note: Agents 1 & 2 timed out for bigSearch, but it was included because Agent 3 worked.
![](img/P2ExecutionTime.png)  
*Figure 6*  
![](img/P2NodesExpanded.png)  
*Figure 7*  
![](img/P2PathCost.png)  
*Figure 8*  

#### Analysis

##### A* agents pale in comparison to Agent 3
Why does Agent 3 seem so much more efficient in every way when compared to Agents 1 & 2? The trade-off lies within the fact that Agent 3 does not always find the shortest path through the maze. The primary objective of the agents that use A* is to find the absolute shortest path to the farthest food from us where we still collect all the food dots on the way there. This always leaves us with the shortest path that checks all the boxes, but at the expense of A*'s exponential time and memory complexity (O(b^d) in both cases). As we saw, this led to runtime problems with large mazes not being able to be solved because of timeout or running out of memory. As intuition would have it, the PacMan problem belongs to the set of NP-Hard problems that get nearly impossible to solve with traditional optimizing algorithms as they increase in size. (Source: This great paper on complexity theory of video games https://arxiv.org/abs/1201.4995) As such, large input mazes fail with the A* method, which essentially checks a whole lot of possible solutions before choosing the optimal one.  
In contrast, the ClosestDotSearchAgent doesn't do that, so it eliminates our concerns about time complexity and the size of mazes on which we are able to operate suddenly is gigantic. However, we sacrifice our reliability here, we may not always get the shortest path through the maze that always touches all the dots. The description in the `findPathToClosestDot` method in the `searchAgents.py` file does a great job of explaining itself: Because the ClosestDotSearchAgent (and the corresponding ClosestDotSearchProblem) is greedy and will always choose the closest dot (using a quick BFS to find that dot), it will sometimes leave a dot all by itself in a part of the maze, and will then have to come all the way back at the end to eat it. This is best observed by running Agent 3 on the bigSearch maze. We can predict with relative certainty that A* agents may have been able to find the shortest path through bigSearch that grabs all the dots, and that the one that Agent 3 found probably isn't the optimal path. But unfortunately the laws of runtime complexity forbid us from having our cake and eating it too.  

##### Comparison of Agents 1 & 2, the A* searches
These two agents are almost identical in how they operate, which is to be expected because they use basically the same algorithm. The only difference between these two is how the heuristic function is measured. Agent 1 uses the Manhattan distance while Agent 2 uses a quantity dubbed "Maze distance" which is like Manhattan distance, but accounts for walls and doesn't count a potential move through a wall as a possible way to get to the goal in the way that Manhattan distance does. This allows us to avoid the situation described above for A* using the Manhattan distance on the userSpirals maze, where A* ends up at a wall that's really close to the goal, but an immovable wall nonetheless.  
The tradeoff between these two is pretty clear from the plots: Agent 2 would be better to run if time is on your side but your memory is smaller, while Agent 1 will get the job done quicker but at the expense of using more memory to expand more nodes. When we input bigger mazes, Agent 1 is more likely to run out of memory first, while Agent 2 will probably take longer and is more likely to time out.  
*Why does a subtle change in heursitic cause this?* Well, clearly the Maze distance is a more admissible heuristic than the Manhattan distance, as it more accurately reflects reality by taking the walls into account, and won't make the mistake of being fooled down the wrong path simply because it is close to the goal (which is the dot furthest away from us at the start). This means it doesn't explore certain nodes that the Manhattan version does. But then, shouldn't it take LESS time, not more? In theory, yes, but the added time comes from the fact that in order to get this super-accurate maze heuristic, we're running our other search algorithms (namely, BFS) on the maze because we need to know where the walls are in the first place. This takes more time because of more calculations involved, but once we know where the walls are, the algorithm itself is fairly quick. Either way, there is still the issue of A* being uneffective on increasingly complex mazes, but for mazes that are smaller both these algorithms do a fine job.  

## Technical Conclusions
When we look at data from running these different algorithms we start to really notice the trends. In most cases, a trade-off exists between different algorithms involving running time, memory consumption, nodes expanded, and whether or not we find the optimal least-cost path. As such, different algorithms are suited for different use cases, and each has its advantages and disadvantages. The goal of this study was not to crown an outright winner for different problems, so we won't here. But to recap, here is a summary of the conclusions I've touched upon in this report.
- BFS will always find you the optimal solution, but takes a bit more memory and time to do so. A trade-off exists between BFS and DFS in terms of performance vs. reliability of finding the optimal solution.
- A* is very powerful for one-food problems, showing the power of the heuristic search, and the Manhattan distance version is extremely effective, showing once again that the heuristic function chosen can have a drastic effect on the outcome and performance of the algorithm.
- Changing the problem or board around can change which algorithm is most effective. Different algorithms can work better on some problems than others. However the heuristic searches edge out blind searches most of the time.
- Searching algorithms can be implemented in terms of the agent itself.
- For large-scale, NP-hard problems, developing an optimal algorithm that doesn't overload performance can be tough. A lot of computing power is needed for algorithms that work by constantly checking most possible solutions, such as A*. There is still some work to be done in the field of Algorithms to be able to solve these efficiently.

## Personal Comments
I really enjoyed working on this project. Looking at problems that have clear solutions like this one and are not too large are a great way to really take a look at common algorithms and pit them against each other. This is the kind of stuff that makes me love studying computer science.  
One challenge I had was spending hours trying to design a maze where the A* with Manhattan distance heuristic would expand more nodes than the BFS. I kept getting closer and closer by continually altering my maze so that BFS and A* would have to explore the exact same number of nodes. Eventually I got them to explore the same number, and I was convinced at that point that becasue of the way the algorithms work, that this was actually impossible. A Google search confirmed my theory. Thank goodness for the Internet. Designing the other maze, where DFS expands less nodes than A*, was also challenging and took a lot of trial and error, but I eventually figured out in what situtations A* would be tricked into spending most of it's time checking out solutions that were close in proximity to the goal, but blocked off by a wall. That part was super rewarding to figure out and was probably my favorite challenge in this project. I feel designing the mazes I had to really understand the algorithms well and it forced me to test things out and see which algorithms did better in which cases, which was almost fun, in a way.  
Overall I really enjoyed this project and would love to do more like it in the future.









