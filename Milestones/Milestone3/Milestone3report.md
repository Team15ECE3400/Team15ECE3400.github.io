# Milestone 3
## Simulation Search Method

For the simulation, we referred to Team Alpha’s Code initially to implement depth first search. First, we ran their GUI and looked at the various example mazes, and then tried to implement a similar version of the simulation in C++. 

The reason we chose C++ was because it was later easier to configure to arduino code while also having a wide array of libraries to use. 

To generate randomized mazes, we first created a global multidimensional array of int’s that stored where the walls of the maze were. In our generate_maze function, we iterated through our map array (with nested for loops), to generate a random integer value between 0 and 16 using the rand() function in C++. We calculated this value by taking one plus rand() mod 1000, and dividing that by zero. Essentially, we the output is a value between 0 and 2 which is then rounded up or down. We add this value four times to generate four random values that then add up to 16. This integer was then stored in the respective location in map. 

The reason we generate a value between 0 and 16 is that we have a 4 bit value which can then be translated into whether there is a wall or not. Basically, each bit represents a defined wall direction (North, South, East, West), where North is the most significant bit and West is the least significant bit. 

For example, if we were in the starting grid, there would definitely be walls at the edges of the map. This means there would be a wall south and west of the robot, and our bit value would be 0101, which corresponds to an integer value of 5. Therefore, 5 would be stored in map[5][4] and then later be extracted and evaluated bitwise. 

![](https://github.com/Team15ECE3400/Team15ECE3400.github.io/blob/master/Milestones/Milestone3/table.png)
>Figure x. Representation of the array as a table

Then, we attempted to implement depth first search. 

The libraries that we relied on were stdio.h (default library), iostream (to be able to print on the console), stack (data structure) and bitset (used to extract bit values from integers). First we defined a struct as such: 

``` 
struct Node {
char visited = 0; //check if visited
int x; //x location
int y; //y location
}; 
```

This Node represents a box on a grid, which was represented by a multidimensional 5x4 array of Nodes. Each Node contains the x and y location of the grid (because the index can’t be extracted if we are not looping over the grid) and a flag to check if the Node has been visited or not. 

To traverse the maze, we first create a stack (last in, first out) to store visited nodes. Then, we create our multidimensional array of nodes and initialize each nodes x, y and visited value (visited = 0). Then, we push the starting Node (maze[5][4]) onto the stack.

Our while loop runs until our visited stack is empty. Here are the steps we take to implement DFS:

> Set current position to the top of the stack (current node you are at)

> Pop the top of the stack and mark that node as visited (update the flag)

> Update the current x and y location from the popped node

> Access the map array that stores wall information using bitset. What this does is takes the value stored in our wall location grid and converts it into a 4 element array of 1 or 0

> We then evaluate the value of each character, which represents a direction (as described above). We check for North, South, East and West if a wall exists (character = 0) and assign it the opposite of if visited. 

> Then, we evaluate if we should go in each direction and if we do, we push onto the visited stack. 

> We continue this process until all nodes are visited and the stack is empty. 

To optimize the code, we decided to think about priority first. In other words, we thought about which direction our robot would travel next, and what factors would influence it.

The priority set by Team Alpha was go North, then East, then West, then South. This traversal only allows the robot to turn left and visit unvisited nodes. However, the robot can detect walls approximately 30 cm in front of it, which means that we can see a wall in front of the robot in an adjacent node. To incorporate this, we added three more priorities: 

> East -> South -> West -> North
> West -> North -> East -> South
> South -> West -> East -> North

We did this by re-organizing and reordering the same code used for the N->E->W->S. To pick and choose which priority is evaluated and added to the stack first, we defined another layer of priorities (N->E->W->S). Then, we extracted the bits of number of walls for each adjacent node to the current node. For example, if we’re at the starting node (5,4), we check if the node one above and one to the side (all the adjacent nodes) have walls opposite the robot. If a wall exists, we add that to our stack and traverse that sequence first. This is so that our robot prioritizes dead ends, which would reduce our net run time. If we do not detect a potential dead end, however, we add the next set.

![_](https://github.com/Team15ECE3400/Team15ECE3400.github.io/blob/master/Milestones/Milestone3/robot%20thought%20process.gif)
>Figure x. A brief demonstration of the dead end test.

This logic, however, has certain bugs. Sometimes, the simulated robot crosses through walls, or gets stuck in an infinite loop. We will be hard at work debugging and optimizing this simulation code!

## Representing the Maze

We attempted to print the entire maze using the console. Though a visual representation of the maze is useful for testing purposes, the result of this is difficult to interpret, and requires to be printed anew every time the robot moves a square. As such, this is an inefficient and unsightly method of presenting our solution. The code we used for the console representation is below, followed by a sample generated maze:

```    for (int i = 0; i<5; i++) {
	    for (int j = 0; j<4; j++) {
		    if (map[i][j].wall_W) { cout << "["; } else { cout << " "; }
		    if (map[i][j].wall_S && map[i][j].wall_N) { cout << "="; }
		    else if (map[i][j].wall_S) { cout << "_"; }
		    else if (map[i][j].wall_N) { cout << "-"; }
		    else { cout << " ";}
		    if (map[i][j].wall_E) { cout << "]"; } else { cout << " "; }
	    }
	    cout << endl;
	}
	cout << endl;

```
```
    _  -   ]                                                                                                             
  ]   [- [                                                                                                                                       
[  [  [                                                                                                                                          
 _     - [                                                                                                                                       
[ ]  ]    =  
```

Because the printing code was not fully functional, we decided to use Team Alpha’s example and optimize it using Matlab’s GUI. 

## Maze Simulation of Algorithm

## Real Life Maze Algorithm
Our first task for the real life algorithm that facilitates maze exploration was to merge line following code, IR wall detecting code, and the algorithm code. The line following and IR wall detecting code was completed in a previous lab and milestone. The algorithm was recently made and is based off of the algorithm used in Matlab simulation.

We first started with merging the IR wall detecting code and algorithm code into the line following code. Line following code was turned into the function LineFollowing() and runs at all times in the main loop except for when the robot approaches a black line crossing. The LineFollowing() function has an if case that performs the logic for line following (discussed in milestone 1) and checks for a black line crossing. If a black line crossing is detected, the maze exploration algorithm traverse() is called. When traverse() is executed, it immediately calls the infrared wall detecting function IR() to determine locations of walls relative to the robot at its current black line intersection. After IR() is completed, the traverse() function continues and updates visited in the maze's array and determines which way to turn the robot based on the locations of walls and the current state of visited nodes. One of the boolean global variables "turn180", "turnRight", "turnLeft", and "goStraight" is assigned 1 in traverse(), determined by the algorithm which is discussed in later paragraph, and when the CPU pointer returns to the main loop(), one of the if statements for controlling manuevers is entered and moves the robot left 90 degrees, right 90 degrees, forward 0 degrees, or turnaround 180 degrees.

The traverse() algorithm is a depth-first search (DFS) algorithm that makes going straight highest priority, turning right second priority, turning left as third priority, and turning around as least priority. A global stack with nodes is used to keep track of visited and is used to tell the robot to go N, S, E, or W. The maze exploration algorithm is heavily based on the matlab version, where the matlab version had the advantage of easily maintaining the four cardinal directions NSEW. However, our robot changes direction often and needs a way to keep track of NSEW, so we had to add logic to maintain the integrity of our heading. We use a global variable "direction" to keep track of which way the robot is facing and used this variable to help the algorithm make decisions. The direction variable takes on four values: 0 is North, 1 is East, 2 is West, and 3 is South. The robot's direction variable is used in conjuction with the variables that tell the robot to go N, S, E, or W in order to decide whether to turn right, left, go straight, or turn around. We created four large if statements for this purpose in traverse(), which accept boolean variables "go_north", "go_east", "go_west", or "go_south".

When the robot has finished exploring the maze, it spins in circles indefinitely. In future weeks we will implement a light or sound signal to indicate that the robot has completed maze exploration.

We were not able to finish troubleshooting this code in lab, so we have no footage to show the potential of our logic. Our robot does navigate on thr black tape and executes turns, but there is more work to be done to mature this code. We have identified our code's issues. Our visited nodes seem to not be communicating consistently with variables used to tell the robot to change direction, so we will need to pinpoint where the communication breakdown occurs. more examples ... ? (need to list at least 3 flaws)


### Main Loop
```
void loop() {

LineFollowing(); // perform line following at all times except when at intersection

  if (turnRight) { // Robot turns 90 degrees right
        left.write(100);
        right.write(100); 
        while(!digitalRead(LeftRear)); // Wait here until left rear sensor senses line
        turnRight = 0; // Perform action once
  } 
  if (turnLeft) {  // Robot turns 90 degrees left
        left.write(80);
        right.write(80);
        while(!digitalRead(RightRear)); // Wait here until right rear sensor senses line
        turnLeft = 0; // Perform action once
  }
  if (turn180) {  // Robot turns 180 degrees in clockwise rotation
        left.write(100);
        right.write(100);
        while(!digitalRead(LeftRear)); // Wait here until left rear sensor senses line
        while(!digitalRead(LeftRear)); // Wait here until left rear sensor senses line second time
        turn180 = 0; // Perform action once
  }
  if (goStraight) { // Robot continues straight
        left.write(100);
        right.write(80);
        while(!digitalRead(LeftRear)); // Wait here until left rear sensor senses line
        goStraight = 0; // Perform action once
  }
}
```
>Figure x. Main loop


### Traverse Algorithm
```
// "direction" variable
// 0 is North
// 1 is East
// 2 is West
// 3 is South

void traverse() {

  IR(); // Check for walls and update boolean global variables "leftWallSensor", "rightWallSensor", and "frontWallSensor"

  Serial.println("Traverse"); 
  visited.push(maze[width][height]); 
  
        Node curr_pos; 
        Node next_pos;
  

  if (!visited.isEmpty()) {
    
    curr_pos = visited.peek(); //current position is peek of stack
    visited.pop(); 
    curr_x = curr_pos.x; 
    curr_y = curr_pos.y;
    curr_pos.visited = 1; //mark as visited
    Serial.print(curr_x); Serial.println(curr_y);
    //Look for next wall to visit
    checkWalls();
    //string wall_bin = bitset<4>(wall_loc[curr_x][curr_y]).to_string(); //to binary
    int wall_bin = wall_loc[curr_x][curr_y]; 
    //check if wall at north
    if (bitRead(wall_bin,0) == 0) go_north = !(maze[curr_x][curr_y+1].visited );
    else go_north = 0; 

    //check if wall at east
    if (bitRead(wall_bin, 1) == 0) go_east = (!(maze[curr_x-1][curr_y].visited & !go_north) );
    else go_east = 0; 

    //check if wall at west
    if (bitRead(wall_bin, 2) == 0) go_west = (!maze[curr_x+1][curr_y].visited & !go_east); 
    else go_west = 0; 

    //check if wall at south
    if (bitRead(wall_bin, 3) == 0) go_south = ((!maze[curr_x][curr_y - 1].visited) & !go_west); 
    else go_south = 0;

  Serial.print(direction);

    if (go_north) {
      next_pos = maze[curr_x][curr_y + 1];
      visited.push(next_pos);
      if (direction == 0) goStraight = 1;
      if (direction == 1) turnLeft = 1;
      if (direction == 2) turnRight = 1;
      if (direction == 3) turn180 = 1;
      
      direction = 0;
      Serial.println("Now go North");
      
    }
    else if (go_east) {
      next_pos = maze[curr_x - 1][curr_y];
      visited.push(next_pos);
      if (direction == 0) turnRight = 1;
      if (direction == 1) goStraight = 1;
      if (direction == 2) turn180 = 1;
      if (direction == 3) turnLeft = 1;
      direction = 1;
      Serial.println("Now go East");
    }
    else if (go_west) {
      next_pos = maze[curr_x - 1][curr_y];
      visited.push(next_pos);
      if (direction == 0) turnLeft = 1;
      if (direction == 1) turn180 = 1;
      if (direction == 2) goStraight = 1;
      if (direction == 3) turnRight = 1;
      direction = 2;
      Serial.println("Now go West");
    }
    else if (go_south) {
      next_pos = maze[curr_x][curr_y - 1];
      visited.push(next_pos);
      if (direction == 0) turn180 = 1;
      if (direction == 1) turnRight = 1;
      if (direction == 2) turnLeft = 1;
      if (direction == 3) goStraight = 1;
      direction = 3;
      Serial.println("Now go South");
    }
    else {
      next_pos = visited.peek(); 
      visited.pop(); 
    }

  }
  else {
    Serial.print("Maze is complete");
    left.write(100);
    right.write(100);
    while(1);
  }
  
}


  void checkWalls(){
  if(direction ==0){
    bitWrite(wall_loc[curr_x][curr_y], 0, frontWallSensor);
    bitWrite(wall_loc[curr_x][curr_y], 1, rightWallSensor);
    bitWrite(wall_loc[curr_x][curr_y], 2, leftWallSensor);
    bitWrite(wall_loc[curr_x][curr_y], 3, 0);
  }
  if(direction ==1){
    bitWrite(wall_loc[curr_x][curr_y], 0, leftWallSensor);
    bitWrite(wall_loc[curr_x][curr_y], 1, frontWallSensor);
    bitWrite(wall_loc[curr_x][curr_y], 2, 0);
    bitWrite(wall_loc[curr_x][curr_y], 3, rightWallSensor);
  }
  if(direction ==2){
    bitWrite(wall_loc[curr_x][curr_y], 2, frontWallSensor);
    bitWrite(wall_loc[curr_x][curr_y], 1, 0);
    bitWrite(wall_loc[curr_x][curr_y], 0, rightWallSensor);
    bitWrite(wall_loc[curr_x][curr_y], 3, leftWallSensor);
  }
  if(direction == 3){
    bitWrite(wall_loc[curr_x][curr_y], 2, rightWallSensor);
    bitWrite(wall_loc[curr_x][curr_y], 3, frontWallSensor);
    bitWrite(wall_loc[curr_x][curr_y], 1, leftWallSensor);
    bitWrite(wall_loc[curr_x][curr_y], 0, 0);
  }
}
```
>Figure x. Maze exploration algorithm
