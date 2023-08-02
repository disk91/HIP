# HIP XX: Update Data Rewards Post HPR Deployment

- Author: @disk91
- Start Date: 2023/07/28
- Category: Economic, Technical
- Tracking issue:
- Voting Requirements: veIoT Holders

# Summary 

The Helium Packet Router (HPR), which has been operational for a few months, has initiated changes that supersede the previous constraint-based rules governing data rewards. The aim of this Improvement Proposal (HIP) is to take advantage of these changes and reconsider the way we distribute data rewards to hotspot owners.

Recognizing that downlink communications - essential for JOIN ACCEPT, DOWNLINK, and ACK - are a crucial and limited resource in a scalable network, our suggestion is to incentivize hotspot owners for managing these tasks. The HIP is suggesting a fee of 2DC per 24-byte block of downlink communications.

To ensure the security of the JOIN process, which is a relatively infrequent procedure, we propose to make the JOIN REQUEST uplink traffic free of charge. This enables all copies to be accounted for, thus enhancing the capacity of devices to join the network. However, the corresponding JOIN ACCEPT response will be charged as previously detailed.

# Motivation

## About Downlinks & Ack

Ack is a Donwlink with no data.

With the arrival of the HPR, it is now possible to reward the downlink communications. Previously, the use of state channel and negociation with the hotspots for creating a transaction was too long to preserve the downlink timing. HPR is working differently and can, without impacting the LoRaWAN timing, reward the donwlink work.

Downlink is a critical and limited resource for a LoRaWAN network for many different reasons:
- Transmission limits applicable to device ( 1% to 10% duty cycle in EU, 400ms max time frequency usage within 10s for FCC ) are also applicable to hotspot when communicating to devices
- Downlink communications stops all reception ability on all the channels for the Hotspot
- Downlink communications, high power, creates long range collisions

The new ability to have Class-B and Class-C devices, with the arrival of the open-LNS solutions, generate the ability to have more downlinks than uplinks on certain solutions, making basically this kind of solution free of charge for them. This means having the hotspot owners are working for free to run these planned or continuous downlinks.

For these reasons, we propose to make the downlink rewarded to the hotspot owners, as the downlink is a critical and limited resource, we propose to reward it 2 DCs per bloc of 24 bytes.

The impact for the hotspot owners is a new type of reward for donwlink traffic currently not retributed.

## About JOIN REQUEST

On LoRaWAN networks, most end-devices utilize a process called Over-The-Air Activation (OTAA) to initialize their connection with the network through a "JOIN". The join procedure consists of two messages exchanged with the LoRaWAN Network Server (LNS), namely a join-request (sent from the device) and a join-accept (sent from the LNS to the device via a Hotspot). On Helium, all of these messages pass through OpenLNS (referred to as Helium Packet Router) in order to appropriately direct and account for network traffic.
JOIN REQUEST is really rare, it comes at the device power-up and may be executed on regular basis to renew the communication keys.
It should be executed at least before 65536 uplinks for security reasons. It does not exist with ABP devices.

This HIP outlines a solution to issues that arise under this model:
- Due to the two-way handshake that must be performed during a JOIN, it is imperative that the LNS knows of the most well-positioned Hotspot in terms of signal strength (RSSI) to deliver the JOIN ACCEPT. In order to do this, Helium Packet Router will need to forward all receipts of the packet reported by Hotspots to ensure the highest opportunity for success.
- Currently, JOIN REQUEST are concidered as standard uplink, the HPR, is selecting the JOIN REQUEST with the same rules as uplink : first comes are selected, up to the limit of max_copies frames.

By addressing this issues, the Helium Network:
- Simplify economics of operating devices on the Helium network by eliminating the unknown cost of a device join.
- Replace the JOIN REQUEST (uplink) cost by a JOIN ACCEPT (downlink) cost as previously mentionned.

The impact of the proposal for the hotpost owner is positive as currently the JOIN REQUEST is rewarded 1DC but with that proposal the JOIN REQUEST + JOIN ACCEPT will be rewarded 2 DCs. The better quality of the network in the JOIN process will help to make the growth of the traffic higher.

## Currentl Volumes to be considered

We don't have official metrics for all the element discussed above, by the way, we can consider the following as a starting point:
- The Helium daily uplink volume, cleared from the non standard activities is about 15.000.000 DCs (from Dune) 
- The Roaming daily uplink volume is about 6.000.000 DCs (from Dune)
- The number of hotspot rewarded per day for data transfer is about 75000 (HeliumGeek)
Assumptions:
- Average uplink packet cost is 1.5 DCs
- The JOIN REQUEST volume is around 1.5% of UPLINK assuming 250.000 a day, assuming 500.000 DCs less burn a day
- The DOWNLINK + JOIN ACCEPT volume is around 15% of UPLINK volume, with x2 DCs per DOWNLINK, assuming 5.500.000 DCs burn a day 

# Rationale and Alternatives

## JOIN Process evolution

Before HIP70, Hotspots passed data to an LNS through a bid mechanism. Hotspots would report to the LNS’s individual router that it had data available and the LNS would have a choice whether to purchase it or not. While this allowed for clear purchase intent, this added additional latency and bandwidth concerns to all packet transfer as well as requiring specialized software (router) to be integrated with the LNS. With the changes presented by HIP70, the data passes directly between Hotspots and LNSs via the unified Helium Packet Router. 

## Downlink rewarding

As hotpost selection for downlink communication is usually based on signal streight, we see here an opportunity of gaming by facking the signal information delivered by the hotspot until the secured concentrator are deployed everywhere (really long term perspective). For this reason, the downlink reward can't be linked to the action of a specific hotspot.
We see two ways for solving this problem:

### Deliver DCs to participating hotspots

The downlink DCs will be given to the hotpost participating to uplink process but not especially to the one executing the downlink. The distribution will depend on the type of requests:
- for a JOIN REQUEST / JOIN ACCEPT, the JOIN ACCEPT DCs will be given to one of the hotposts having relayed the JOIN REQUEST packet. This hotspot is selected ranomly in the hotspot list, after the JOIN ACCEPTS has been received by the HPR.
- for a Downlink / Ack, the DCs will be distributed, after the downlink has been received by the HPR to the hotpost thay relays the uplink at first. As most of the uplink have a max_copy of 1, this is also correspondng to the one executing the downlink.

This approach, is close to the reality of the work but it creates a certain complexity in the HPR and it can generate slowness as the HPR must store the uplink packets and make the link with the later comming downlink paquets. We are not recommanding this approach if Nova concider it as dangerous or hard to implement.

### Manage a pool of downlink DCs to be distributed on regular basis

This second approach is to create a pool of reward composed by the downlink DCs. Any downlink have a cost of 2DCs for the OUI but these DCs are not directly given to the hostpots involved in these communications. They are saved in a dedicated DC pool. On every day, the DC pool is distributed equaly, to all the hotspots having realyed data traffic during this period of time.

As a DC can't be split, the number of DCs must be greated than the number of hotspot rewarded to have this distribution occuring. Otherwize, the DCs are kept in pool and the distribution will be made in a later occurence. For the same reason, the DCs that can't be distributed, will be distributed dring the next run. 

With about 70K hostpots transfering data a day and a volume of DCs from downlink pool about 5M, most of the DCs pool can be distributed on every days.

# Stakeholders

### Hotspot owners

Hotspots are rewarded for the data they transfer at a fixed value of $0.00001 (remunerated in the network token). By excluding device JOIN_REQUEST by adding JOIN_ACCEPT rewarding and DOWNLINK & ACK rewarding the global volume of reward may increased after the HIP as estimated in the volume to be concidered. Given that Hotspot Hosts represent a high percentage of voting power, their voting participation will strongly signal their position as a stakeholder.

### Roaming partner

To be investigated

### LNS Owners

LNS operators are rewarded for processing traffic. They also need to address user connectivity issues. The selection of Hotspots in downlink communication is a critical factor for success, so providing LNS operators with the ability to choose the best option can reduce support-related queries. 
As representatives of device owners, LNS operators can offer a better service to device fleet owners by simplifying the join procedure and enabling them to maintain a low 'max_copy' parameter, thus keeping network usage costs in-line with delivered service. Service providers can benefit from this by achieving a smoother connection process, even with 'max_copy' set to 1 for subsequent messages. This helps them avoid high costs when a poorly developed device initiates a join loop.

# Unresolved Questions

### Unsuccessfull downlink

Downlinks will be rewarded 2DCs to the hotspot but we have no solution to make sure that this downlink has been a success. Potentially an unsuccessful downlink will have a cost for the device owner with no guaranty of result.

# Deployment Impact

This requires
- an evolution of the HPR to manage the new rewarding mechanism, it impacts Nova developpement team.
- an evolution of the legacy console to manage the device owner DC balance in regard of the new rewarding mechanism, it impacts Nova development team.
- potential evolution of the open-LNS to Helium integration layers to manage the device owner DC balance, like for legacy console, it impacts third party developments.
- Cost for device owner is becomming higher if they were using downlink intensively. But the get a better JOIN ability.

# Success Metrics

- Better ability for devices to JOIN the network
- Higher Data transfer reward for hotspot owner
- Long terme reduction of the downlink communications

