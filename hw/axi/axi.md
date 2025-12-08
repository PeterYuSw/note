# AXI

## basic
- signal
	- input signal sampled on the rising edge of ACLK
	- output signal can only occur after the rising edge of ACLK

- channel: each channel transfers information in only one direction
	- AW
	- W
	- AR
	- R
	- B

## valid-ready handshake
- VALID must remained asserted until handshake occurs
	- at a rising clock edge when VALID and READY are both asserted
- **transfer happens at the rising clock edge when VALID and READY are both asserted**

## order
- AXI read returning in order from transaction that use same ID

## transaction
- transaction must not cross a 4KB boundary

