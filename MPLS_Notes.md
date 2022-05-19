Juniper:

### Labeled Switched Path:

- LSP -> LDP - Lable Distribution Protocol
	- Floods LSPs on all interfaces(MPLS enabled)
	- By defauld advertises LSPs for /32 interfaces routes
	- Default routing Source is IGP
	- Default Route Preference(AD) is 9 
		
- RSVP - Resource Reservation Protovol
	- Floods LSPs only along the path
	- None default advertisments of any routes
	- Default Routing Source CSPF <-- Constrained Shortest Path First. The routing using CSPF is known as Constraint Based Routing (CBR).
	  A constraint could be minimum bandwidth required per link (also known as bandwidth guaranteed constraint), end-to-end delay, maximum number of links traversed, include/exclude nodes.
	- Default AD is 7
		
		
### Configuration Hierarchy:

#### RSVP: 
```
[edit protocols rsvp] 
interface ge-0/0/0.0;
```
#### LDP: 
```
[edit protocols mpls]
 labeled-switched-path R7_to_R2 {
		to 10.210.0.2;
 }
 
Disable CSPF(Optional):
	  [edit protocols mpls]
	  no-cspf;
```	
#### Troubleshooting Commands:
```
	show rsvp neighbor
	show rsvp interface
	show rsvp statistics 
	show route protocol rsvp
	show rsvp session
	show rsvp session egress detail | match path.*rcvfrom | match ae216 | count
	
	
	show mpls lsp [terse | brief | detail | extensive | statistics | etc.]
	show route hidden  [terse | brief | detail | extensive ]
	show route resolution unresolved
	
	show route 192.168.1.0/25 extensive | match reason
                Inactive reason: Not Best in its group - Router ID
                Inactive reason: Not Best in its group - Router ID
                Inactive reason: Not Best in its group - Router ID
```	