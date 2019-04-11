# Finite-state Machine (FSM)

* Finite-state machines (FSM)
    * Idle
        * Refuses all incoming BGP connections
        * Starts initialization of event triggers
        * Initiates TCP session with configured BGP peers
        * Listens for TCP connection from peer
        * Changes state to Connect
        * If an error occurs, session terminated and state returns to Idle. Reasons include:
            * TCP port 179 is not open
            * Random TCP port over 1023 isn't open
            * Peer address configured incorrectly on either router
            * AS number configured incorrectly on either router
    * Connect
        * Waits for successful TCP negotiation
        * Sends Open message to peer and changes state to OpenSent
        * If error occurs, changes to Active state
    * Active
        * Router was unable to establish a successful TCP session
        * BGP FSM tries to restart a TCP session with peer, if successful, sends Open message
        * If it is unsuccessful again, FSM reset to Idle. Reasons include:
            * TCP port 179 not open
            * Random port over 1023 not open
            * BGP config error
            * Network congestion
            * Flapping interface
    * OpenSent
        * Listens for an Open message from peer
        * Once Rxed, router checks validity of open message
        * If an error occurs, it's because one of the fields in the Open message does not match
            * BGP version mismatch
            * Expects a different AS
        * If there is no error, Keepalive is sent and various timers are set. FSM changes to OpenConfirm
    * OpenConfirm
        * Peer is listening for a Keepalive from its peer
        * If Keepalive is Rxed and no timers have expired FSM moves to Established
        * If there is an error, router transitions back to Idle
    * Established
        * Peers send update messages to exchange information about routes being advertised to the BGP peer
        * If there is any error in the Update message, notification message is sent to the peer and BGP transitions back to Idle
        * If a time expires before a Keepalive is Rxed or an error occurs, router transitions back to Idle
