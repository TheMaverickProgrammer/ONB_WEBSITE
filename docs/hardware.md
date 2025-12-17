# TOPICS
Since its conception, ONB has been developed with low-end and embedded devices in mind
as a first-class deployment target.

ONB Client can be built with configurations to reduce the resource requirements needed
to run smoothly on less performant hardware. Additional programs can cross-communicate with the Client to expand on the experience.

Choose one of the hardware options below which best describes your interest with ONB integration to learn more.

## Options
1. [ONB Link Gate](./hardware_docs/link_gate.md)

# EXTERNAL DEVICES
ONB Client listens to an external hardware's IP address by the flag `-x`. This flag stands for E`x`ternal Device.

Any remote program can be an External Device and it uses a fairly simple protocol.

!!! note
    Protocols are only available for 2.5+ clients.

## Packets
The packets are identified by a leading byte.

```cpp
/* Utility [0x01 - 0x05] */
u_draw_group_flags = 0x01,
u_frame_skip,
u_frame_resume,
u_debug_origin,
u_log_level,

/* Gamepad [0x06 - 0x11] */
gp_left,
gp_right,
gp_up,
gp_down,
gp_lshoulder,
gp_rshoulder,
gp_select,
gp_start,
gp_btn_a,
gp_btn_b,
gp_btn_x,
gp_btn_y,

/* Game Elements [0x12 - 0x18] */
ge_set_reg_mem,
ge_set_player_health,
ge_set_player_max_health,
ge_set_player_speed_level,
ge_set_player_charge_level,
ge_set_player_attack_level,
ge_set_player_money,

/* Chip Gate [0x19] */
cg_send_uuid,

/* Response Codes [0x1A - 0x1B] */
rc_log_data,
rc_frame_status,
```

!!! note
    The packet is identified by the External Device Packet Processor in a background thread - or if running ONB Client single-threaded: before the next frame paints to the screen.

### Utilities

The utility packets are identified by leading byte values `0x01` to `0x05` respectively.

#### Values for `u_draw_group_flags`

todo

#### Values for `u_log_level`

<center>
<pre>
<-------- 16 bytes --------->
|             |             |
|  8-bit ID   | 8-bit value |
|             |             |
</pre>
</center>

todo

#### Values for `u_debug_origin`

A 16-bit packed RGB color value. A non-zero color will display and colorize origin cross-hairs on all sprite nodes rendered on screen. This is useful for debugging graphics.

<center>
<pre>
<-------- 16 bytes --------->
|          |         |      |
|  3b R    | 3b G    | 2b B |
|          |         |      |
</pre>
</center>


#### Values for `u_frame_skip`

Any non-zero 16-bit value is evaluated to `true` for this event and ONB Client will stop advancing the internal clock. This is useful for debugging the client. Every subsequent `u_frame_skip` packet will advance the client's frame by one.

#### Values for `u_frame_resume`

Any non-zero 16-bit value is evaluated to `true` for this event and ONB Client will resume the internal clock normally.

### Gamepad

The gamepad packets are identified by leading byte values `0x06` through `0x11` respectively.

These packets have the following anatomy:

<center>
<pre>
<--------------- 24 bytes ---------------->
|             |             |             |
|  8-bit ID   |         16-bit value      |
|             |             |             |
</pre>
</center>

#### Values

Any non-zero 16-bit value is evaluated to `true` for this event and ONB Client will transform the value into a button `pressed` event. Consecutive packets will prolong the `pressed` event and the Client will transform those events to `held` events. 

Once these packets are no longer received, the Client will transform these buttons into one frame of `released` events.

### Game Elements

The game element packets are identified by leading byte values `0x12` to `0x18` respectively.

These packets have the following anatomy:

<center>
<pre>
<--------------- 24 bytes ---------------->
|             |             |             |
|  8-bit ID   |         16-bit value      |
|             |             |             |
</pre>
</center>

#### Values

The 16-bit value will be applied directly to the Client's active Player Session for that field. This is useful for debugging custom games or severs.

### ONB Link Gate

The Link Gate packets are identified by the byte `0x19`.

These packets have the following anatomy:

<center>
<pre>
<-------------- 8 + N bytes --------------->
|             |                            |
|  8-bit ID   | N-bit null-terminated str  |
|             |                            |
</pre>
</center>

That is, the `value` can be variable in size.

#### Values

The `value` part of the packet is variable in size because strings can be of arbitrary length. That is to say the multi-byte character string is read to the end of the packet and interpretted as a chip ID.

If the Client has the corresponding chip mod loaded in memory, a `Card` construct will be placed in the net navi's hand during Battle.

### Response Codes

The response code packets are identified by leading byte values `0x1A` to `0x1B` respectively.

These packets are send _from_ Client to the connected External Device. Therefore, the Client will never _read_ these packets.

#### Values for `rc_log_data`

This packet contains log data to display inside the ONB Inspector tool.

#### Values for `rc_frame_data`

This packet contains battle frame data for auditing battles and debugging possible desyncs.
