![ScotMesh Reticulum](https://raw.githubusercontent.com/ScotMesh/branding/main/networks/reticulum/readme-header.png)

# RNS over Meshtastic — ScotMesh fork

> **This is a fork of [landandair/RNS_Over_Meshtastic](https://github.com/landandair/RNS_Over_Meshtastic), maintained by ScotMesh for the gateway at The Pole, Cadham.** The upstream documentation follows below, unchanged; this box lists what is different.
>
> **New interface options**, all optional:
>
> | Option | What it does | Default |
> |---|---|---|
> | `channel_index` | Sends and receives the tunnel on this Meshtastic channel slot instead of the primary channel (0). Put the tunnel on its own secondary channel so it doesn't share the public channel's key and traffic. Packets on other channels are ignored. | `0` |
> | `announce_max_hops` | Drops outgoing **announces** that have already travelled more than this many Reticulum hops. On a transport node with a backbone, this stops the whole route table being re-announced onto a shared LoRa channel, while still allowing a mode such as `gateway`. Path responses are exempt, so path lookups still work. | off |
> | `fragment_size` | Bytes of Reticulum data per Meshtastic packet. Gateways that pass frames through meshtasticd's simulated radio, such as RepeaterTastic, take at most 230 bytes of encrypted data. meshtasticd 2.8 signs a broadcast (+66 bytes) whenever it still fits a LoRa frame, so packets of 158–166 bytes end up too big once signed; 155 keeps every packet signed and within the limit. Fragments of different sizes still reassemble, so senders and receivers may differ. | `155` (upstream: 200) |
>
> `hop_limit` is also clamped to 0–7, since Meshtastic stores it in 3 bits.
>
> ```ini
> [[Meshtastic Interface]]
>   type = Meshtastic_Interface
>   enabled = yes
>   mode = gateway
>   port = /dev/ttyUSB0
>   data_speed = 0            # LongFast
>   channel_index = 1         # the tunnel's own channel
>   hop_limit = 7
>   announce_max_hops = 2     # transport nodes only
> ```
>
> How to join ScotMesh's tunnel: [Reticulum over MeshCore and Meshtastic](https://wiki.scotmesh.net/wiki/Reticulum_over_MeshCore_and_Meshtastic). The MeshCore counterpart is [ScotMesh/RNS_Over_Meshcore](https://github.com/ScotMesh/RNS_Over_Meshcore).

# RNS_Over_Meshtastic
Interface for RNS using Meshtastic as the underlying networking layer to utilize existing meshtastic hardware.

- This is a direct followup project to https://github.com/landandair/Meshtastic_File_Transfer and fixes many of its issues (use the rncp utility of RNS and get a more reliable easy to use utility)
- Consider the expected max speed to be around 500 bytes/s so notably worse than RNode
- has the benefit of being propagated and functional alongside existing standalone Meshtastic Nodes
- Ideal use case would be as a faster secondary meshtastic network covering an area providing a route for more intensive data uses such as RNS

## Usage
- Install Meshtastic Python Library
- Add the file [Meshtastic_Interface.py](Interface%2FMeshtastic_Interface.py) to your interfaces folder for reticulum
- Modify the node config file and add the following
```
 [[Meshtastic Interface]]
  type = Meshtastic_Interface
  enabled = true
  mode = gateway
  port = /dev/[path to device]  # Optional: Meshtastic serial device port
  ble_port = short_1234  # Optional: Meshtastic BLE device ID (Replacement for serial port)
  tcp_port = 127.0.0.1:4403  #Optional: Meshtastic TCP IP. [port is optional if using default port] (Replacement for serial or ble)
  data_speed = 8  # Radio speed setting desired for the network(do not use long-fast)
```

- Radio settings and their associated transfer speeds are shown below; time unit is seconds between packets (from [Meshtastic_Interface.py](Interface%2FMeshtastic_Interface.py))
```python
speed_to_delay = {8: .4,  # Short-range Turbo (recommended)
                  6: 1,  # Short Fast (best if short turbo is unavailable)
                  5: 3,  # Short-range Slow (best if short turbo is unavailable)
                  7: 12,  # Long Range - moderate Fast
                  4: 4,  # Medium Range - Fast  (Slowest recommended speed)
                  3:6,  # Medium Range - Slow
                  1: 15,  # Long Range - Slow
                  0: 8  # Long Range - Fast
                  }
```
- Use the number on the left to set the speed and change the number on the right to change the max transmission rate from the radio
