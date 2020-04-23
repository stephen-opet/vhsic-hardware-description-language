An extension of the seven-segment LED decoder - when compiled, this VHDL program iterates through values 0-F, reseting and repeating infinitely.

The program was built to run on a Digilent Nexys A7-100T board - the software files included here encompass a Vivado project

Core files include those from the led decoder project (leddec.vhd, leddec.xdc), as well as these files:

hexcount.vhd, a VHDL source file:
```
LIBRARY IEEE;
USE IEEE.STD_LOGIC_1164.ALL;

ENTITY hexcount IS
	PORT (
		clk_50MHz : IN STD_LOGIC;
		anode : OUT STD_LOGIC_VECTOR (3 DOWNTO 0);
		seg : OUT STD_LOGIC_VECTOR (6 DOWNTO 0)
	);
END hexcount;

ARCHITECTURE Behavioral OF hexcount IS
	COMPONENT counter IS
		PORT (
			clk : IN STD_LOGIC;
			count : OUT STD_LOGIC_VECTOR (3 DOWNTO 0)
		);
	END COMPONENT;
	COMPONENT leddec IS
		PORT (
			dig : IN STD_LOGIC_VECTOR (1 DOWNTO 0);
			data : IN STD_LOGIC_VECTOR (3 DOWNTO 0);
			anode : OUT STD_LOGIC_VECTOR (3 DOWNTO 0);
			seg : OUT STD_LOGIC_VECTOR (6 DOWNTO 0)
		);
	END COMPONENT;
	SIGNAL S : STD_LOGIC_VECTOR (3 DOWNTO 0);
BEGIN
	C1 : counter
	PORT MAP(clk => clk_50MHz, count => S);
	L1 : leddec
	PORT MAP(dig => "00", data => S, anode => anode, seg => seg);
END Behavioral;
```

count.vhd, a VHDL source file:
```
LIBRARY IEEE;
USE IEEE.STD_LOGIC_1164.ALL;
USE IEEE.STD_LOGIC_UNSIGNED.ALL;

ENTITY counter IS
	PORT (
		clk : IN STD_LOGIC;
		count : OUT STD_LOGIC_VECTOR (3 DOWNTO 0)
	);
END counter;

ARCHITECTURE Behavioral OF counter IS
	SIGNAL cnt : STD_LOGIC_VECTOR (28 DOWNTO 0); -- 29 bit counter
BEGIN
	PROCESS (clk)
	BEGIN
		IF clk'EVENT AND clk = '1' THEN -- on rising edge of clock
			cnt <= cnt + 1; -- increment counter
		END IF;
	END PROCESS;
	count <= cnt (28 DOWNTO 25);
END Behavioral;
```

and hexcount.xdc, a Vivado constraint file:
```
#FPGA I/0 Locations

set_property -dict {PACKAGE_PIN E3 IOSTANDARD LVCMOS33} [get_ports clk_50MHz]

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
```
