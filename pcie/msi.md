# MSI

## basic
- PCIe must support MSI or MSI-X interrupt

- trigger: EP write ```message data``` to ```message address```
	- defined in MSI/MSI-X capability
	- memory write TLP

- MSI
	- max 32 continuous vector
	- message data defined by cpu
	- same message address?
	- mask bit & pending bit -> avoid interrupt drop

- MSI-X
	- 2048 **non-continuous**
	- edge-triggered
	- MSI-X table
		- independent message data and message address
		- init by bios in PCIe emuration
		- stored in BAR space
		- multi entries in table
			- one vector per entry
			- |per vector mask|msg data|msg upper addr|msg addr|
	- capability
		- table BIR: bar index
		- table offset: offset in BAR
	- pending bit table
		- PBA(pending bit array) bir
		- PBA offset
		- per bit per vector, per entry 64 vectors


