This Bouncing Ball program was built through Vivado VHDL on the XILINX Digilent A7-100T FPGA board. The project uses five VHDL source files (clk_wiz_0.vhd, clk_wiz_0_clk_wiz.vhd, vga_sync.vhd, ball.vhd, and vga_top.vhd) and one Vivado constraint file (vga_top.xdc) to generate a game of pong on a 800x600 Video Graphics Array (VGA) monitor.

Once synthesizing, implementing and programming the FPGA with the baseline program, a red, square pong ball of size 8 will be shown through the VGA output monitor, bouncing back and forth on a single vertical axis.

The size of our pong ball can be modified by editing the code snippet included here, found just within the architecture body declaration. The pong ball's 'radius' is defined by the constant value of 'size' - altering the constant value of this constant from its initial value of 8 to 32 produces a pong ball quadrupled in size:
```
ARCHITECTURE Behavioral OF ball IS
--	CODE TO CHANGE RADIUS OF BALL
-- 	CONSTANT size  : INTEGER := 8; 
--	    ^ INITIAL CODE, BALL SIZE 8
	CONSTANT size  : INTEGER := 32;
--	    ^ NEW CODE, BALL SIZE 32
```


Likewise, the RGB colors of our VGA output can be changed by manipulating the steady state of three standard logic output signals for the 'ball' entity: 'red', 'green', and 'blue'. These ports are declared in the entity declaration for our 'ball'. Though the ball was initially loaded to red, one can change this ball color by editing the value of these output signals; this is accomplished within the architecture body, just below the signal declaration section (shown below). These signals can take one of four discrete states: 1 (implying this color is active for all screen pixels), 0 (implying this color is off for all screen pixels), ball_on (a signal from the architecture declarations, containing ball pixels), and NOT ball_on (every pixel not contained within the ball). Initially, red was set to be on for all screen pixels, while green and blue were on for all non-ball pixels - the result was a red ball on a white background. This code has been reconfigured to turn green on for ball pixels, blue on for non-ball pixels, and red off completely - the result should be a green ball on a blue background:
```
BEGIN -- Architecture Body
--  EDIT RGB COLOR OUTPUT

--  INITIAL CODE  
-- 	red 	<=	'1';  	
--	green 	<=	NOT ball_on;
-- 	blue 	<=	NOT ball_on;  

--  NEW CODE 
	red 	<=	'0'; 
	green 	<=	ball_on;
	blue  	<=	NOT ball_on;
```

Changing the shape of our pong ball requires a deep dive into the bdraw process, a process defined within the body of our 'ball' architecture responsible for drawing the pong ball per frame. As addressed above in "Change Size of Pong Ball", there is a constant within the architecture called 'size', which defines the radius of our pong ball. Likewise, the "Manipulate RGB Color Output" section addresses a signal called 'ball_on', a boolean that is true if a pixel is within the boundary of our ball. As shown in the commented-out snippet of code, the initial program draws a square by taking the current position of the ball (pixel_col,pixel_row), and encompassing any pixels plus or minus the value of our radius 'size' within ball_on. To draw a circle with the same radius, this computation must be replaced by the equation for a circle, x^2 + y^2 = r^2. Positions for x and y can be computed by taking the difference between the pixel in question, pixel_col/row, and the center of the ball, ball_x/y (since we are taking squared values, negative values are inconsequential). To avoid VGA weirdness, each constituent value must be cast to an integer before evaluation with the IEEE function CONV_INTEGER().
```
-- process to draw ball 
-- current pixel address is covered by ball position
bdraw : PROCESS (ball_x, ball_y, pixel_row, pixel_col) IS
BEGIN
    --INITIAL CODE | DRAWS A SQUARE
    --    IF    (pixel_col >= ball_x - size) 
    --    AND (pixel_col <= ball_x + size) 
    --    AND (pixel_row >= ball_y - size) 
    --    AND (pixel_row <= ball_y + size) 
    --    THEN ball_on <= '1';
	
    --use distance formula to create a circular pong ball
    --values must be cast to integers or weird ugliness follows
	-- sqrt(x^2 + y^2) = r
	-- => (column-x)^2 + (row-y)^2 = const_size^2
    IF ( ((CONV_INTEGER(pixel_col) - CONV_INTEGER(ball_x))  	
        *(CONV_INTEGER(pixel_col)-CONV_INTEGER(ball_x)))    
      + ((CONV_INTEGER(pixel_row)-CONV_INTEGER(ball_y))     
	*(CONV_INTEGER(pixel_row)-CONV_INTEGER(ball_y)))    
      <= size*size) THEN			         
	ball_on <= '1';
    ELSE
        ball_on <= '0';
    END IF;
END PROCESS;
```

Finally, we add a horizontal axis to the pong ball's motion by making two changes to the ball.vhd source code. First of all, a new signal, named 'ball_x_motion', must be defined within the architecture declaration section - this is shown below in the architecture's signal declaration. Note the default value, '00000000000', refers to the initial velocity of the ball when bouncing in the respective axis.
```
-- indicates whether ball is over current pixel position
SIGNAL ball_on : STD_LOGIC; 

-- current ball position - intitialized to center of screen
SIGNAL ball_x  : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(400, 11);
SIGNAL ball_y  : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(300, 11);

-- current ball motion - initialized to +4 pixels/frame
SIGNAL ball_y_motion : STD_LOGIC_VECTOR(10 DOWNTO 0) := "00000000100";

--Adds horizontal motion to the ball
SIGNAL ball_x_motion : STD_LOGIC_VECTOR(10 DOWNTO 0) := "00000000100";
```

Once a signal has been added for the x-axis motion of our ball in architecture declarations, the signal behavior must be defined in the 'mball' process, wherein the ball's movement per frame is delineated. The upper section defines the ball's movement along the vertical axis, while the bottom section defines the ball's new movement along the horizontal axis. The value of 800 is specific to the monitor I am using, and may need to be customized according to the dimensions of the VGA monitor this program is ran on.
```
-- process to move ball once every frame (i.e. once every vsync pulse)
mball : PROCESS
BEGIN
	WAIT UNTIL rising_edge(v_sync);
	-- allow for bounce off top or bottom of screen
	--Replace '600' with specific monitor dimensions
	IF ball_y + size >= 600 THEN
		ball_y_motion <= "11111111100"; -- -4 pixels
	ELSIF ball_y <= size THEN
		ball_y_motion <= "00000000100"; -- +4 pixels
	END IF;
	ball_y <= ball_y + ball_y_motion; -- compute next ball position

	--Added for horizontal ball motion
	--Replace '800' with specific monitor dimensions
	IF ball_x + size >= 800 THEN
		ball_x_motion <= "11111111100"; -- -4 pixels
	ELSIF ball_x <= size THEN
		ball_x_motion <= "00000000100"; -- +4 pixels
	END IF;
	ball_x <= ball_x + ball_x_motion;	
END PROCESS;
```
