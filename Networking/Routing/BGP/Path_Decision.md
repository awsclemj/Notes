- [Path Decision Algorithm](#path-decision-algorithm)
  - [Definitions](#definitions)
  - [Algorithm](#algorithm)
  - [Attribute Specifics](#attribute-specifics)

# Path Decision Algorithm
## Definitions
* AS Path - An ordered list of all the autonomous systems through which this update has passed. Mandatory.
* Origin - How BGP learned of this network
    * i = by network command, e = from EGP, ? = redistributed from other source
        * Mandatory
* Local Preference -  value telling iBGP peers which path to select for traffic leaving the AS
    * Default is 100
    * Discretionary
* Multi-exit Discriminator (MED) - suggest to a neighboring autonomous system which of multiple paths to select for traffic bound into your AS
    * Lowest MED is preferred
    * Optional, non-transitive

## Algorithm
1. Choose the route with highest weight (Cisco specific)
2. If weight is not set, choose route with highest local preference
3. Choose routes that this router originated
4. Choose the path with the shortest AS path
5. Choose path the lowest origin code (I is lowest, e is next, ? Is last)
6. Choose route with the lowest MED, if the same AS advertises the possible routes
7. Choose eBGP over iBGP
8. Choose route through the nearest IGP neighbor as determined by the lowest IGP metric
9. Choose oldest route
10. Choose path through the neighbor with the lowers router ID
11. Choose path through the neighbor with the lowest IP address

## Attribute Specifics
* Weight
    * Cisco-specific attribute
    * Local to the router it’s configured
    * Highest weight is preferred
* Local preference
    * Attribute for controlling egress traffic (outgoing)
    * Tells iBGP peers which path to prefer
    * Valid only within the AS
    * Higher value preferred
* AS Path
    * Prepending - advertising the ASN multiple times in order to bias the traffic ingress
    * This is how you can tell external ASes that you want to prefer a certain path
    * Prepends include several instances of the ASN and advertise these additional ASNs
* MED
    * Sets relative preference for ingress traffic
    * Non-transitive (only between adjacent AS)
    * Good example of load balancing (can LB between ISPs, for instance)
    * Lower value preferred
        * 10.0.1.0/24 MED 10 is preferred over MED 20
        * Local preference overrides this value
