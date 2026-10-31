# Distance Vector Routing in Cisco Packet Tracer

A three-router network that uses RIP version 2 to show distance vector routing. Each router starts out knowing only its directly connected networks and learns the rest from its neighbours, with no static routes.

Built for the Computer Networks course, Semester V.

## Open the project

Open `Distance_Vector_Routing.pkt` in Cisco Packet Tracer 9.0.1 or later.

## Topology

```
LAN 1 192.168.1.0/24       LAN 2 192.168.2.0/24       LAN 3 192.168.3.0/24
  PC0   PC1                  PC2   PC3                  PC4   PC5
    \   /                      \   /                      \   /
   Switch0                    Switch1                    Switch2
      |                          |                          |
   Router0 ---- 10.0.0.0/8 --- Router1 ---- 11.0.0.0/8 --- Router2
```

All routers are `Router-PT` and all switches are `Switch-PT`. The routers are connected by serial links.

## Addressing

| Device  | Interface       | IP address   | Mask          | Default gateway |
|---------|-----------------|--------------|---------------|-----------------|
| Router0 | FastEthernet0/0 | 192.168.1.1  | 255.255.255.0 | n/a             |
| Router0 | Serial2/0       | 10.0.0.1     | 255.0.0.0     | n/a             |
| Router1 | FastEthernet0/0 | 192.168.2.1  | 255.255.255.0 | n/a             |
| Router1 | Serial2/0 (DCE) | 10.0.0.2     | 255.0.0.0     | n/a             |
| Router1 | Serial3/0       | 11.0.0.1     | 255.0.0.0     | n/a             |
| Router2 | FastEthernet0/0 | 192.168.3.1  | 255.255.255.0 | n/a             |
| Router2 | Serial2/0 (DCE) | 11.0.0.2     | 255.0.0.0     | n/a             |
| PC0     | FastEthernet0   | 192.168.1.2  | 255.255.255.0 | 192.168.1.1     |
| PC1     | FastEthernet0   | 192.168.1.3  | 255.255.255.0 | 192.168.1.1     |
| PC2     | FastEthernet0   | 192.168.2.2  | 255.255.255.0 | 192.168.2.1     |
| PC3     | FastEthernet0   | 192.168.2.3  | 255.255.255.0 | 192.168.2.1     |
| PC4     | FastEthernet0   | 192.168.3.2  | 255.255.255.0 | 192.168.3.1     |
| PC5     | FastEthernet0   | 192.168.3.3  | 255.255.255.0 | 192.168.3.1     |

## RIP networks

| Router  | Networks advertised             |
|---------|---------------------------------|
| Router0 | 10.0.0.0, 192.168.1.0           |
| Router1 | 10.0.0.0, 11.0.0.0, 192.168.2.0 |
| Router2 | 11.0.0.0, 192.168.3.0           |

Each router also has `version 2` and `no auto-summary` set under `router rip`, and its LAN port is a passive interface, so RIP advertises the LAN but sends no updates to the PCs. Every router and switch uses its device name as its hostname, and its configuration is saved to the startup configuration.

## Simulations

The file includes five saved scenarios. Choose one from the scenario list at the bottom right.

| Scenario         | Pings                              | Route                                 |
|------------------|------------------------------------|---------------------------------------|
| 1 Same LAN       | PC0 to PC1, PC2 to PC3, PC4 to PC5 | The switch only, no router            |
| 2 LAN 1 to LAN 2 | PC0 to PC2 and back                | Router0 and Router1, metric 1         |
| 3 LAN 2 to LAN 3 | PC2 to PC4 and back                | Router1 and Router2, metric 1         |
| 4 LAN 1 to LAN 3 | PC0 to PC4 and back                | All three routers, metric 2           |
| 5 All LANs       | PC1 to PC3, PC3 to PC5, PC5 to PC1 | Every router pair, metric 1 or 2      |

To run one:

1. Click **Fast Forward Time** a few times so RIP converges.
2. Choose a scenario.
3. Switch to **Simulation** mode. The event filters are set to show only RIP and ICMP.
4. Click **Fire** next to a ping in the PDU list at the bottom right.
5. Press **Play**, or **Capture/Forward** to step through one event at a time. Click an envelope to inspect a packet.

The PDU list should show **Successful**. If the first attempt fails while ARP resolves, fire it again.

RIP updates appear only on the serial links, because the LAN ports are passive interfaces.

### Simulate a failure

To watch RIP react to a lost network, run these commands on Router2:

```
enable
configure terminal
interface FastEthernet0/0
shutdown
```

Router2 tells its neighbours that LAN 3 is gone. After a short wait, `show ip route` on Router0 should no longer show 192.168.3.0/24 as reachable, and pings from PC0 to 192.168.3.2 should fail.

Run `no shutdown` on the same interface to restore it. Don't save the file while the interface is shut down.

## Test the routing

Wait about a minute for RIP to converge, then run these checks.

From the command prompt on PC0, ping a host in each of the other LANs:

```
ping 192.168.2.2
ping 192.168.3.2
tracert 192.168.3.2
```

The first ping can time out while ARP resolves. The trace should pass through 192.168.1.1, 10.0.0.2 and 11.0.0.2 before reaching PC4.

On each router, run `show ip route`. These are the routes the configuration should produce:

**Router0**

```
C    10.0.0.0/8 is directly connected, Serial2/0
R    11.0.0.0/8 [120/1] via 10.0.0.2, Serial2/0
C    192.168.1.0/24 is directly connected, FastEthernet0/0
R    192.168.2.0/24 [120/1] via 10.0.0.2, Serial2/0
R    192.168.3.0/24 [120/2] via 10.0.0.2, Serial2/0
```

**Router1**

```
C    10.0.0.0/8 is directly connected, Serial2/0
C    11.0.0.0/8 is directly connected, Serial3/0
R    192.168.1.0/24 [120/1] via 10.0.0.1, Serial2/0
C    192.168.2.0/24 is directly connected, FastEthernet0/0
R    192.168.3.0/24 [120/1] via 11.0.0.2, Serial3/0
```

**Router2**

```
R    10.0.0.0/8 [120/1] via 11.0.0.1, Serial2/0
C    11.0.0.0/8 is directly connected, Serial2/0
R    192.168.1.0/24 [120/2] via 11.0.0.1, Serial2/0
R    192.168.2.0/24 [120/1] via 11.0.0.1, Serial2/0
C    192.168.3.0/24 is directly connected, FastEthernet0/0
```

In `[120/2]`, 120 is RIP's administrative distance and 2 is the hop count. The real output also shows a route timer, omitted here.
