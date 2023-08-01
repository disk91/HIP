# HIP XX: Update Data Rewards Post HPR Deployment

- Author: @disk91
- Start Date: 2023/07/28
- Category: Economic, Technical
- Tracking issue:
- Voting Requirements: veIoT Holders

# Summary 

The Helium Packet Router (HPR) deployed since a couple of month changed the previous constraints driven some of the rules of the data rewards.
The purpose of the HIP is to get benefit of this evolution to think the data reward to hostpot owner a bit differently.

As the downlink communications, required for JOIN ACCEPT, DOWNLINK and ACK, are a critical and limited resource in a scalable network, we propose 
to reward the hotspot owners for processing it. The HIP proposes a cost of 2DC per 24 Bytes blocs of downlink communications.

As we need to secure the JOIN procedure and as this process is rare, we propose that JOIN REQUEST uplink traffic will be free of charge. This will allow to take all the copies in consideration to improve the ability of devices to JOIN the network. (The JOIN ACCEPT response will be charged as indicated previously).

# Motivation

## About Downlinks & Ack

Ack is a Donwlink with no data.

With the arrival of the HPR, it is now possible to reward the downlink communications. Previously, the use of state channel and negociation with the 
hotspots for creating a transaction was too long to preserve the downlink timing. HPR is working differently and can, without impacting the LoRaWAN timing, reward the donwlink work.

Downlink is a critical and limited resource for a LoRaWAN network for many different reasons:
- Transmission limits applicable to device ( 1% to 10% duty cycle in EU, 400ms max time frequency usage within 10s for FCC ) are also applicable to hotspot when communicating to devices
- Downlink communications stops all reception ability on all the channels for the Hotspot
- Downlink communications, high power, creates long range collisions

The new ability to have Class-B and Class-C devices, with the arrival of the open-LNS solutions, generate the ability to have more downlinks than uplinks on certain solutions, making basically this kind of solution free of charge for them. This means having the hotspot owners are working for free to run these planned or continuous downlinks.

For these reasons, we propose to make the downlink rewarded to the hotspot owners, as the downlink is a critical and limited resource, we propose to reward it 2 DCs per bloc of 24 bytes.

The impact for the hotspot owners is a new type of reward for donwlink traffic currently not retributed.

## About JOIN REQUEST

The ability for a device to join the network ( mandatory process for OTAA communications ) with a JOIN REQUEST is related to ability to find the best hotspot for sending the downlink response for the JOIN ACCEPT. For this, the LNS will select the hotspot with the signal with the best matching. The best matching rules depends on the LNS algorithm, in many case it takes the best signal.

Currently, JOIN REQUEST are concidered as standard uplink, the HPR, is selecting the JOIN REQUEST with the same rules as uplink : first comes are selected, up to the limit of max_copies frames. 

JOIN REQUEST is really rare, it comes at the device power-up and may be executed on regular basis to renew the communication keys.
It should be executed at least before 65536 uplinks for security reasons. It does not exist with ABP devices.

The current way to secure the ability for a device to connect to the newtork is to increase the max_copies parameter, it means that all the future uplinks will cost up to max_copies DCs per blocs of 24 bytes. For a good ability to connect, you should have a max_copies of 5 or more having for consequence to multiply all your communications cost by 5. Practically speaking, most of the business solution won't have a such setup and will be impacted by the difficulty to join the network. As a consequence, Helium is not percieved at the right level of quality, due to this recurrent connectivity issue.

The proposal is to make the JOIN REQUEST packet free of charge, as a consequence, hotspot owner won't be rewarded for this type of packet. The HPR will transfer all the packets to the LNS and that one will be able to select the best of them. This one will get rewarded through the JOIN ACCEPT downlink packet as previously proposed.

The impact of the proposal for the hotpost owner is positive as currently the JOIN REQUEST is rewarded 1DC but with that proposal the JOIN REQUEST + JOIN ACCEPT will be rewarded 2 DCs. The better quality of the network in the JOIN process will help to make the growth of the traffic higher.

# Rationale and Alternatives

As hotpost selection for downlink communication is usually based on signal streight, we see here an opportunity of gaming by facking the signal information delivered by the hotspot until the secured concentrator are deployed everywhere (really long term perspective). For this reason, the downlink reward can't be linked to the action of a specific hotspot.
We see two ways for solving this problem:

### Deliver DCs to participating hotspots

The downlink DCs will be given to the hotpost participating to uplink process but not especially to the one executing the downlink. The distribution will depend on the type of requests:
- for a JOIN REQUEST / JOIN ACCEPT, the JOIN ACCEPT DCs will be given to one of the hotposts having relayed the JOIN REQUEST packet. This hotspot is selected ranomly in the hotspot list, after the JOIN ACCEPTS has been received by the HPR.
- for a Downlink / Ack, the DCs will be distributed, after the downlink has been received by the HPR to the hotpost thay relays the uplink at first. As most of the uplink have a max_copy of 1, this is also correspondng to the one executing the downlink.

This approach, is close to the reality of the work but it creates a certain complexity in the HPR and it can generate slowness as the HPR must store the uplink packets and make the link with the later comming downlink paquets. We are not recommanding this approach if Nova concider it as dangerous or hard to implement.

### Manage a pool of downlink DCs to be distributed on regular basis

This second approach is to create a pool of reward composed by the downlink DCs. Any downlink have a cost of 2DCs for the OUI but these DCs are not directly given to the hostpots involved in these communications. They are saved in a dedicated DC pool. On every week, the DC pool is distributed equaly, to all the hotspots having realyed data traffic during this period of time.

As a DC can't be split, the number of DCs must be greated than the number of hotspot rewarded to have this distribution occuring. Otherwize, the DCs are kept in pool and the distribution will be made in a later occurence. For the same reason, the DCs that can't be distributed, will be distributed dring the next run.


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

