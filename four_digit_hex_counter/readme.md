This VHDL program uses four LED segment displays to iteratively tally 65536 values; the counter uses hexidecimal values, and counts from 0000 to FFFF. 

This project is built, synthesized and implemented on XLINIX Vivado. The program is executed and built for a Diligent Nexys A7-100T board

The project uses four VHDL source files:

leddec.vhd:
```
LIBRARY IEEE;
USE IEEE.STD_LOGIC_1164.ALL;

ENTITY leddec IS
	PORT (
		dig : IN STD_LOGIC_VECTOR (1 DOWNTO 0);
		data : IN STD_LOGIC_VECTOR (3 DOWNTO 0);
		anode : OUT STD_LOGIC_VECTOR (3 DOWNTO 0);
		seg : OUT STD_LOGIC_VECTOR (6 DOWNTO 0)
	);
END leddec;

ARCHITECTURE Behavioral OF leddec IS
BEGIN
	-- Turn on segments corresponding to 4-bit data word
	seg <= "0000001" WHEN data = "0000" ELSE --0
	       "1001111" WHEN data = "0001" ELSE --1
	       "0010010" WHEN data = "0010" ELSE --2
	       "0000110" WHEN data = "0011" ELSE --3
	       "1001100" WHEN data = "0100" ELSE --4
	       "0100100" WHEN data = "0101" ELSE --5
	       "0100000" WHEN data = "0110" ELSE --6
	       "0001111" WHEN data = "0111" ELSE --7
	       "0000000" WHEN data = "1000" ELSE --8
	       "0000100" WHEN data = "1001" ELSE --9
	       "0001000" WHEN data = "1010" ELSE --A
	       "1100000" WHEN data = "1011" ELSE --B
	       "0110001" WHEN data = "1100" ELSE --C
	       "1000010" WHEN data = "1101" ELSE --D
	       "0110000" WHEN data = "1110" ELSE --E
	       "0111000" WHEN data = "1111" ELSE --F
	       "1111111";
	-- Turn on anode of 7-segment display addressed by 2-bit digit selector dig
	anode <= "1110" WHEN dig = "00" ELSE --0
	         "1101" WHEN dig = "01" ELSE --1
	         "1011" WHEN dig = "10" ELSE --2
	         "0111" WHEN dig = "11" ELSE --3
	         "1111";
END Behavioral;
```

counter.vhd:
```
-- counter.vhd --

LIBRARY IEEE;
USE IEEE.STD_LOGIC_1164.ALL;
USE IEEE.STD_LOGIC_UNSIGNED.ALL;

ENTITY counter IS
	PORT (
		clk : IN STD_LOGIC;
		count : OUT STD_LOGIC_VECTOR (15 DOWNTO 0); --NEED REVISE! 16 bits
		mpx : OUT STD_LOGIC_VECTOR (1 DOWNTO 0)); --NEW ONE ADD! send signal to select displays
END counter;

ARCHITECTURE Behavioral OF counter IS
	SIGNAL cnt : STD_LOGIC_VECTOR (38 DOWNTO 0); -- 39 bit counter
BEGIN
	PROCESS (clk)
	BEGIN
		IF clk'EVENT AND clk = '1' THEN -- on rising edge of clock
			cnt <= cnt + 1; -- increment counter
		END IF;
	END PROCESS;
	count <= cnt (38 DOWNTO 23); -- 16bits
	mpx <= cnt (18 DOWNTO 17); -- 2bits
END Behavioral;
```

hexcount.vhd:
```
-- hexcount.vhd --

LIBRARY IEEE;
USE IEEE.STD_LOGIC_1164.ALL;

ENTITY hexcount IS
	PORT (
		clk_100MHz : IN STD_LOGIC;
		anode : OUT STD_LOGIC_VECTOR (3 DOWNTO 0);
		seg : OUT STD_LOGIC_VECTOR (6 DOWNTO 0)
	);
END hexcount;

ARCHITECTURE Behavioral OF hexcount IS

	COMPONENT counter IS
		PORT (
			clk : IN STD_LOGIC;
			count : OUT STD_LOGIC_VECTOR (15 DOWNTO 0); --NEED CHANGE! counter now output 16 bits for all 4 displays
			mpx : OUT STD_LOGIC_VECTOR (1 DOWNTO 0)
		);
	END COMPONENT;

	COMPONENT leddec IS
		PORT (
			dig : IN STD_LOGIC_VECTOR (1 DOWNTO 0);
			data : IN STD_LOGIC_VECTOR (3 DOWNTO 0); --DONT change, data is fixed 4 bits in leddec for each displays
			anode : OUT STD_LOGIC_VECTOR (3 DOWNTO 0);
			seg : OUT STD_LOGIC_VECTOR (6 DOWNTO 0)
		);
	END COMPONENT;

	SIGNAL S : STD_LOGIC_VECTOR (15 DOWNTO 0); -- connect C1 and L1 for values of 4 digits
	SIGNAL md : STD_LOGIC_VECTOR (1 DOWNTO 0); -- mpx selects displays
	SIGNAL display : STD_LOGIC_VECTOR (3 DOWNTO 0); -- send digit for only one display to leddec

BEGIN
	C1 : counter
	PORT MAP(clk => clk_100MHz, count => S, mpx => md);
	L1 : leddec
	PORT MAP(dig => md, data => display, anode => anode, seg => seg);
	--mpx
	--process(md)
	--begin
	-- if md = "00" then
	-- display <= S(3 downto 0);
	-- elsif md = "01" then
	-- display <= S(7 downto 4);
	-- elsif md = "10" then
	-- display <= S(11 downto 8);
	-- elsif md = "11" then
	-- display <= S(15 downto 12);
	-- end if;
	--end process;

	display <= S(3 DOWNTO 0) WHEN md = "00" ELSE
	           S(7 DOWNTO 4) WHEN md = "01" ELSE
	           S(11 DOWNTO 8) WHEN md = "10" ELSE
	           S(15 DOWNTO 12);

END Behavioral;
```
The project also uses one constraint file, hexcount.xdc:
```
#FPGA I/0 Locations

set_property -dict {PACKAGE_PIN E3 IOSTANDARD LVCMOS33} [get_ports clk_100MHz]

set_property -dict {PACKAGE_PIN L18 IOSTANDARD LVCMOS33} [get_ports {seg[0]}]
set_property -dict {PACKAGE_PIN T11 IOSTANDARD LVCMOS33} [get_ports {seg[1]}]
set_property -dict {PACKAGE_PIN P15 IOSTANDARD LVCMOS33} [get_ports {seg[2]}]
set_property -dict {PACKAGE_PIN K13 IOSTANDARD LVCMOS33} [get_ports {seg[3]}]
set_property -dict {PACKAGE_PIN K16 IOSTANDARD LVCMOS33} [get_ports {seg[4]}]
set_property -dict {PACKAGE_PIN R10 IOSTANDARD LVCMOS33} [get_ports {seg[5]}]
set_property -dict {PACKAGE_PIN T10 IOSTANDARD LVCMOS33} [get_ports {seg[6]}]

set_property -dict {PACKAGE_PIN U13 IOSTANDARD LVCMOS33} [get_ports {anode[0]}]
set_property -dict {PACKAGE_PIN K2 IOSTANDARD LVCMOS33} [get_ports {anode[1]}]
set_property -dict {PACKAGE_PIN T14 IOSTANDARD LVCMOS33} [get_ports {anode[2]}]
set_property -dict {PACKAGE_PIN P14 IOSTANDARD LVCMOS33} [get_ports {anode[3]}]

set_property CONFIG_MODE SPIx1 [current_design]

```
