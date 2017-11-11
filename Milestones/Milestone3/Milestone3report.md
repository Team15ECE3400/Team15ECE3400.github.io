# Milestone 3

## Maze Simulation of Algorithm

## Real Life Maze Algorithm
Our first task for the real life algorithm that facilitates maze exploration was to merge line following code, IR wall detecting code, and the algorithm code. The line following and IR wall detecting code was completed in a previous lab and milestone. The algorithm was recently made and is based off of the algorithm used in Matlab simulation.

We first started with merging the IR wall detecting code and algorithm code into the line following code. Line following code was turned into the function LineFollowing() and runs at all times in the main loop except for when the robot approaches a black line crossing. The LineFollowing() function has an if case that checks for a black line crossing, and if detected, runs the maze exploration algorithm traverse(). When the traverse() function is called, it calls the infrared wall detecting function IR() to determine locations of walls relative to the robot. After IR() is completed, the traverse() function updates visited in the maze's array and determines which way to turn the robot based on the locations of walls and the current state of visited nodes. One of the boolean global variables "turn180", "turnRight", "turnLeft", and "goStraight" is assigned 1 in traverse() and when the CPU pointer returns to the main loop(), one of the if statements for controlling manuevers is entered and moves the robot left 90 degrees, right 90 degrees, forwards 0 degrees, or turnaround 180 degrees.

The traverse() algorithm is a depth-first search (DFS) algorithm that makes going straight highest priority, turning right second highest priority, and turning left as lowest priority. A global stack with nodes is used to keep track of visited, and is used to tell the robot to go N, S, E, or W. The maze exploration algorithm was heavily based on the matlab version, where matlab version had advanted of easily maintaining the four cardinal directions NSEW. However, our robot changes direction often, so we had to add logic to maintain the integrity of our heading. We use a global variable "direction" to keep track of which way the robot is facing and use this variable to help the algorithm make decisions. The direction variable takes on four values: 0 is North, 1 is East, 2 is West, and 3 is South. The direction variable is used in conjuction with the variables used to tell the robot to go N, S, E, or W to decide whether to turn right, left, go straight, or turn around. We created for large if statedments for this purpose.

When the robot has finished exporing the maze, it spins in circles indefinitely. In future weeks we will implement a light or sound signal to indicate that the robot has completed maze exploration.

We were not able to finish troubleshooting this code in lab, so we have no footage to show the potential of our logic. We have identified our code's issues however, which include inability to ... ? (need to list at least 3 flaws)


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
