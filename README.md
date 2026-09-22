# SLD1010 / SIM7100 RFID Manager Settings Guide

> **Purpose:** This document explains the visible RFID Manager settings in simple language, including **what each setting means, when to use it, and practical recommendations** for an industrial RFID application.

---

## 1. Important Notes Before Changing Settings

1. Change one setting at a time.
2. Save/apply the configuration and restart the reader if required.
3. Test the reader after every major change.
4. Keep a backup or screenshot of the original configuration.
5. The exact behavior of some settings can depend on the SIM7100 firmware and RFID Manager version.
6. Do not change RF, network, or tag-access settings randomly in a live production line.

---

# 2. Configuration Overview

The RFID Manager contains two major configuration areas:

| Configuration area | Main purpose |
|---|---|
| Static Parameter Configuration | Network, antenna power, frequency region, serial settings, inventory settings, and tag-operation settings |
| Advanced Parameter Configuration | Data format, upload behavior, event subscriptions, buffering, HTTP upload, GPI/GPO behavior, and advanced inventory options |

---

# 3. Data Format and Tag Field Options

## 3.1 Format Type

**Visible value:** `New format`

### What it means

Defines the structure of the data that the reader sends to an external system.

The format controls which fields are included in the uploaded tag message.

### When to use

Use a format that your server, Raspberry Pi program, database, or PLC integration can parse reliably.

### Recommendation

Use one fixed format during development and production. Avoid changing the format without updating the receiving software.

---

## 3.2 Tag Field Options

These checkboxes determine which data fields are included in the tag report.

| Field | Meaning | When it is useful |
|---|---|---|
| Additional Data | Extra tag information, depending on configuration | When extra tag memory or additional values are required |
| Antenna Number | Identifies the antenna that detected the tag | Very important for identifying the bucket/zone |
| Read Count | Number of times the tag was read or reported | Useful for evaluating repeated reads |
| Frequency | Frequency used for the read | Useful for RF testing and diagnostics |
| Tag Protocol | RFID protocol/type information | Useful for troubleshooting and tag compatibility |
| RSSI | Received Signal Strength Indicator | Useful for signal-strength analysis and zone testing |
| Reserve Field | Reserved or vendor-defined field | Usually leave disabled unless required |
| First Timestamp | Time of the first read in a grouped record | Useful for measuring entry/start time |
| Last Timestamp | Time of the last read in a grouped record | Useful for measuring exit/end time |

### Important for the industrial jig application

Enable at least:

- **Antenna Number**
- **First Timestamp**
- **Last Timestamp**
- **Read Count**
- **RSSI**
- **Tag Protocol**, if required for diagnostics

The first and last timestamps can help estimate how long a tag was visible to the antenna. They do **not automatically guarantee the exact physical dipping time** unless the antenna coverage and reader configuration have been tested.

---

# 4. Advanced Data Upload Options

## 4.1 Client Confirmation

**Visible value:** `No Confirmation Message Sent`

### What it means

Controls whether the reader expects a confirmation/acknowledgement from the receiving client after data is uploaded.

### Possible operating concepts

- **No confirmation:** The reader sends data without waiting for an application-level acknowledgement.
- **Confirmation enabled:** The reader may wait for a response from the server before continuing or clearing data, depending on firmware behavior.

### When to use

| Situation | Suggested approach |
|---|---|
| Simple test server | No confirmation may be sufficient |
| Critical data collection | Use confirmation only after verifying the protocol and retry behavior |
| Network with frequent interruptions | Test confirmation, retry, and queue behavior carefully |

> Confirm the exact acknowledgement format in the SIM7100/SLD1010 protocol manual before enabling a confirmation mode.

---

## 4.2 Response Timeout Duration

**Visible value:** `10`

### What it means

The time the reader waits for a response from the receiving system. The unit must be confirmed from the firmware manual; it is commonly seconds in reader software, but do not assume this without documentation.

### When to use

- Increase it when the server is slow.
- Decrease it only when the network and server are consistently fast.
- Avoid very long timeouts if the system must recover quickly from a disconnected server.

### Recommendation

Start with the default value and measure actual server response time before changing it.

---

## 4.3 Binary Message CRC

**Visible value:** `With crc`

### What it means

CRC (Cyclic Redundancy Check) is used to detect errors in binary messages.

### When to use

- Use CRC when the communication protocol supports it.
- The sender and receiver must use the same CRC method.
- Do not enable CRC on one side and disable it on the other unless the protocol explicitly supports that combination.

### Recommendation

Keep CRC enabled when using a supported binary protocol, especially over noisy industrial communication links.

---

## 4.4 Old Data Cleanup Time

**Visible value:** `8`

### What it means

Controls when old stored or queued data is cleaned up. The exact unit and cleanup conditions must be confirmed in the firmware documentation.

### When to use

This is relevant when the reader stores data during network interruptions.

### Risk

If cleanup occurs too early, old tag records may be deleted before the Raspberry Pi or server receives them.

### Recommendation

Set this only after understanding:

- Where data is stored
- How long the reader can buffer it
- Whether the data is deleted after successful upload
- What happens when storage becomes full

---

# 5. Event Subscription

The event subscription checkboxes determine which types of events the reader sends to the configured destination.

| Event | Meaning | When to enable |
|---|---|---|
| Tag Data | Sends tag-read records | Enable for RFID data collection |
| Heartbeat | Sends periodic status messages | Enable for connection monitoring |
| GPI Change | Reports input-pin changes | Enable when an external sensor or trigger is connected |
| Empty Data | Reports empty/no-data events | Enable only if the application needs them |
| Tag Enter | Reports when a tag enters a defined detection condition | Use only after verifying the reader's enter/exit logic |
| Time Synchronization | Reports time synchronization events | Enable when time synchronization status is needed |

## Recommended event selection for a Raspberry Pi gateway

Start with:

- **Tag Data: Enabled**
- **Heartbeat: Enabled**
- **GPI Change: Disabled**, unless a sensor is connected
- **Empty Data: Disabled**, unless required
- **Tag Enter: Disabled**, unless the application uses this event
- **Time Synchronization: Enable only when required for time monitoring**

---

# 6. Reader Logical Name

**Visible example:** `0826ae114c77`

### What it means

A logical name used to identify the reader in software or uploaded data.

### Rules shown in the interface

- Use ASCII characters.
- Do not include `/`.
- Maximum length: 128 characters.

### Recommended naming format

Use a meaningful industrial name, for example:

```text
PLANT1_LINE1_JIG01_READER01
```

### Why it matters

When several RFID readers send data to one Raspberry Pi server or central database, the logical name helps identify the source of every record.

---

# 7. Heartbeat Cycle

**Visible value:** `10`

### What it means

Defines how often the reader sends a heartbeat/status message.

The interface indicates a valid range of **5–7200 seconds**, with a default of 10 seconds when blank.

### When to use

Heartbeat messages are useful for detecting:

- Network disconnection
- Reader failure
- Server communication problems
- Device online/offline status

### Recommendation

For industrial monitoring:

- 10 seconds: Fast status monitoring
- 30–60 seconds: Lower network traffic
- Longer intervals: Suitable when immediate failure detection is not necessary

Use a value that matches the required response time of the monitoring system.

---

# 8. Send Buffer Length

**Visible value:** `1580`

### What it means

Defines the size of the outgoing communication buffer.

The interface indicates a valid range of **1500–8192 bytes**, with a default of 1580 bytes when blank.

### When to use

A larger buffer may help when:

- A message contains many fields
- Several tag records are grouped together
- The network sends larger packets

### Important

This is **not necessarily the same as permanent storage capacity**. A send buffer is temporary communication memory. It should not be treated as guaranteed offline data storage.

---

# 9. Custom Parameter

### What it means

Allows a custom parameter string to be added to the reader's data or communication configuration.

The interface indicates:

- ASCII characters only
- Maximum length: 64 characters

### When to use

Use it for application-specific information if the firmware and receiving server support it.

Example:

```text
LINE=1;JIG=3
```

### Recommendation

Use a documented format and make sure the server knows how to interpret it.

---

# 10. Network Timeout Restart Time

**Visible value:** `0`

### What it means

Controls automatic restart behavior after a network timeout.

The interface indicates a range of **0–65535 minutes** and states that the default value is 0.

### General interpretation

- `0`: Usually disabled or no automatic restart
- Non-zero value: Enables a timeout-based restart according to the firmware logic

### When to use

Use this only if the reader must automatically recover from a persistent network problem.

### Recommendation

Do not enable it immediately. First confirm whether the restart can interrupt tag collection or cause data loss.

---

# 11. Abnormal Restart Time

**Visible value:** `0`

### What it means

Controls automatic restart behavior after an abnormal condition.

The interface indicates a range of **0–65535 minutes** and states that the default value is 0.

### When to use

Use it only after testing the reader's behavior during:

- Network failure
- Communication timeout
- Firmware/application abnormality
- Repeated disconnections

### Caution

Automatic restarts may temporarily stop RFID reading. Verify buffering and recovery behavior before using this in production.

---

# 12. Initial Count Condition

**Visible value:** `Inventory`

### What it means

Defines the initial condition or counting behavior used by the reader when inventory starts.

### When to use

This may affect how the reader initializes inventory counting or tag-read statistics.

### Recommendation

Keep the default setting unless the application requires a specific counting behavior. Verify the exact meaning in the firmware manual.

---

# 13. Remote Command Recovery

**Visible value:** `Not Reply`

### What it means

Defines how the reader behaves when a remote command does not receive a reply or when command recovery is required.

### When to use

This is relevant when a server or Raspberry Pi sends commands to the reader remotely.

### Recommendation

Test command timeout and recovery behavior before using it in a live system. The exact available modes should be interpreted using the vendor protocol documentation.

---

# 14. Required Parameters

## 14.1 Data Processing Cycle

**Visible value:** `2000`

The interface indicates a valid range of **50–86400000 ms**.

### What it means

Controls the interval used by the reader to process or group tag data before uploading it.

If the value is in milliseconds:

```text
2000 ms = 2 seconds
```

### Practical effect

A longer cycle may:

- Group more reads into one message
- Reduce the number of uploads
- Increase reporting delay

A shorter cycle may:

- Send data more frequently
- Reduce reporting delay
- Increase communication traffic

### Recommendation for jig monitoring

Test values such as:

- 500 ms
- 1000 ms
- 2000 ms

Choose the value based on the required timing accuracy and the number of repeated tag reads.

> The exact grouping behavior must be confirmed using real tag data from the reader.

---

## 14.2 Hardware Interface

**Visible value:** `WIFI`

### What it means

Selects the physical communication interface used for data transfer.

### Common choices

- Wi-Fi
- Ethernet
- Serial interfaces, depending on firmware

### When to use

- Use **Wi-Fi** when the reader is connected through a wireless network.
- Use **Ethernet** when a stable wired connection is available.
- Use serial communication when integrating directly with a controller or computer through RS232/RS485.

### Recommendation

For industrial environments, wired Ethernet is often easier to troubleshoot and can be more stable than Wi-Fi, but the final choice depends on site conditions.

---

## 14.3 Upload Protocol

**Visible value:** `HTTP`

### What it means

Defines the protocol used to upload tag data to the configured server.

### HTTP upload

The reader sends data to an HTTP endpoint such as:

```text
http://10.26.229.101:12345/reader
```

### Important

The receiving server must:

- Listen on the correct IP address and port
- Accept the HTTP method used by the reader
- Accept the expected request path
- Parse the incoming body format
- Return a valid response if the reader expects one

---

# 15. HTTP Parameters

## 15.1 Upload URL

**Visible example:**

```text
http://10.26.229.101:12345/reader
```

### URL breakdown

| Part | Meaning |
|---|---|
| `http://` | Communication protocol |
| `10.26.229.101` | IP address of the receiving server |
| `12345` | TCP port |
| `/reader` | HTTP endpoint/path |

### Example architecture

```text
SIM7100 / SLD1010
        |
        | HTTP upload over Wi-Fi
        v
Raspberry Pi
IP: 10.26.229.101
Port: 12345
Endpoint: /reader
        |
        v
RFID processing script / database
```

### Troubleshooting checklist

On the Raspberry Pi, verify:

```bash
ip addr
```

Check whether the IP address is correct.

Check whether the server is listening:

```bash
ss -tulnp | grep 12345
```

Test the endpoint locally:

```bash
curl -i http://127.0.0.1:12345/reader
```

Check the server logs while presenting a tag.

### Common problems

- Incorrect Raspberry Pi IP address
- Wrong port number
- Server bound only to `127.0.0.1`
- Incorrect HTTP method
- Wrong endpoint path
- Firewall blocking the port
- Reader and Raspberry Pi on different networks
- Server returning an unsupported response

---

# 16. GPI Trigger Operation

**Visible status:** Disabled

### What it means

GPI means **General-Purpose Input**.

A GPI input can receive a signal from an external device such as:

- Proximity sensor
- Limit switch
- PLC output
- Push button
- Jig-position sensor

### When to use

Use GPI trigger operation when RFID reading should start or stop based on an external machine signal.

### Example

```text
Jig sensor detects rod position
        |
        v
SLD1010 GPI input changes state
        |
        v
Reader starts or stops the configured operation
```

### Settings

| Setting | Meaning |
|---|---|
| Trigger Mode | Defines how the input activates the operation |
| Direction | Defines the active signal transition or direction |
| Duration / Reverse Duration | Defines the operation duration or reverse timing, depending on selected mode |

### Recommendation

For an industrial jig, a physical position sensor may be more reliable than using RFID visibility alone to identify the exact start and end of dipping.

---

# 17. GPI Status 1 and GPI Status 2

### What they mean

These sections allow input status conditions to be configured.

The interface shows GPI1, GPI2, GPI3, and GPI4 options with enable checkboxes.

### When to use

Use these settings when different input states must produce different actions or event messages.

### Example use cases

- Jig at home position
- Jig fully dipped
- Jig raised
- Emergency stop status
- Conveyor running status

### Caution

The exact electrical behavior must be checked:

- Input voltage range
- Active-high/active-low logic
- Pull-up/pull-down behavior
- Isolation
- Common ground requirements

Do not connect an industrial voltage directly to a GPIO input without verifying the board's electrical specifications.

---

# 18. Ethernet Settings

## 18.1 IP Address

**Visible value:** `192.168.1.100`

### What it means

The fixed network address of the reader on the Ethernet network.

### When to use

Use a static IP when the reader must always be reachable at the same address.

### Recommendation

For multiple readers:

```text
Reader 1: 192.168.1.100
Reader 2: 192.168.1.101
Reader 3: 192.168.1.102
```

Use addresses that are valid for the actual network and do not conflict with other devices.

---

## 18.2 Subnet Mask

**Visible value:** `255.255.255.0`

### What it means

Defines which devices belong to the same local network.

For a typical `/24` network, devices such as:

```text
192.168.1.100
192.168.1.101
```

are generally in the same subnet.

---

## 18.3 Gateway

**Visible value:** `192.168.1.254`

### What it means

The gateway is the router used to reach other networks.

### When to use

A gateway is required when the reader must communicate outside its local subnet.

---

## 18.4 DNS

**Visible value:** `223.6.6.6`

### What it means

DNS translates domain names into IP addresses.

### When to use

DNS is useful when the reader communicates with a hostname instead of a fixed IP address.

For a local Raspberry Pi server using an IP address, DNS may not be required.

---

## 18.5 DHCP

**Visible value:** Disabled

### What it means

DHCP automatically assigns network settings from a DHCP server/router.

| Mode | Benefit | Risk |
|---|---|---|
| DHCP enabled | Easy installation | IP address may change |
| DHCP disabled/static IP | Predictable address | Manual configuration required |

### Recommendation

For a production RFID system, use either:

- Static IP addresses, or
- DHCP reservations configured in the network router

Avoid duplicate IP addresses.

---

## 18.6 MAC Address

### What it means

A MAC address is the hardware-level network identifier of the interface.

### When to use

Useful for:

- DHCP reservations
- Network identification
- Troubleshooting
- Tracking individual readers

Do not assign the same MAC address to multiple devices.

---

# 19. Wireless Settings

## 19.1 Work Pattern

**Visible value:** `STA Pattern`

### Meaning

STA means **Station mode**. The reader connects to an existing Wi-Fi network.

### Common modes

| Mode | Meaning |
|---|---|
| STA | Connect to an existing Wi-Fi router or hotspot |
| AP | Reader creates its own Wi-Fi access point |
| AP+STA | Reader provides an access point and also connects to another Wi-Fi network, if supported |

### Recommendation

Use **STA mode** when the reader must connect to the plant Wi-Fi or a dedicated router.

---

## 19.2 Wireless IP Address

**Visible value:** `0.0.0.0`

### Meaning

The current wireless IP address is not manually configured or has not yet been assigned.

When DHCP is enabled, the router normally assigns the IP address.

---

## 19.3 Wireless Gateway

**Visible value:** `0.0.0.0`

### Meaning

The wireless gateway has not been manually configured or has not yet been assigned.

---

## 19.4 SSID

**Visible value:** `OPPO`

### Meaning

SSID is the Wi-Fi network name.

### Recommendation

For production, use a dedicated industrial or plant-approved Wi-Fi network instead of a personal hotspot.

---

## 19.5 Wireless DHCP

**Visible value:** Enabled

### Meaning

The wireless interface receives its IP address automatically.

### Recommendation

Use DHCP reservation if the server needs to reach the reader at a predictable address.

---

## 19.6 Wireless Password

The password shown in a configuration screen should be treated as sensitive.

### Recommendation

- Change default or temporary passwords.
- Do not share passwords in screenshots or documentation.
- Use a strong password.
- Store credentials securely.

---

# 20. Antenna Power

**Visible value:** `2000`

### What it means

Controls the RF transmission power setting for the antenna.

The exact unit is firmware-specific. The value may represent a scaled power value rather than a direct dBm number.

### Important

Do not assume:

```text
2000 = 20.00 dBm
```

unless the vendor documentation confirms the scaling.

### When to adjust antenna power

Increase power only when:

- Tags are not detected reliably
- The antenna needs more read range
- Testing shows that higher power improves detection

Decrease power when:

- Tags outside the desired zone are being detected
- Multiple antennas interfere with one another
- Nearby tags are being read unintentionally
- RF exposure or site constraints require lower power

### Industrial recommendation

Start with a moderate value and test:

1. Detection distance
2. False reads outside the target bucket
3. Read consistency
4. Multiple-tag behavior
5. Temperature and long-duration operation

---

# 21. Tag Data Cache Settings

## 21.1 Antenna Uniqueness

### Meaning

Controls whether tag records are considered unique based on antenna information.

### When to use

Enable it when the application needs to distinguish the same tag being detected by different antennas.

### Example

```text
Tag A detected by Antenna 1
Tag A detected by Antenna 2
```

These may need to be treated as separate zone events.

### Caution

The exact uniqueness logic must be verified because it may interact with bank uniqueness and tag timeout settings.

---

## 21.2 Bank Data Uniqueness

**Visible status:** Enabled

### Meaning

Controls uniqueness based on tag memory bank data, depending on firmware behavior.

### When to use

Useful when the system reads additional tag memory and needs to distinguish records based on bank-related information.

### Recommendation

Keep enabled only if it matches the intended tag-deduplication logic. Test with repeated reads and multiple antennas.

---

## 21.3 Record the Highest RSSI

**Visible status:** Disabled

### Meaning

When enabled, the system may retain the highest RSSI value for a tag during a reporting period.

### When to use

Useful when selecting the strongest observed signal during repeated reads.

### Limitation

RSSI should not be treated as an exact distance measurement. Metal, antenna orientation, tag orientation, reflections, and interference can change RSSI.

---

# 22. Working Region

**Visible value:** `North America`

### What it means

Selects the RFID regulatory region and its supported frequency behavior.

### Critical recommendation

Set the working region according to the **actual country where the equipment is installed** and the applicable RFID regulations.

For an installation in India, do not use North America settings without confirming that they are legally and technically appropriate.

### Why it matters

The region can affect:

- Allowed frequencies
- Frequency hopping
- Transmit power limits
- Regulatory compliance

Always confirm the correct region with the device supplier and local regulations.

---

# 23. Frequency Hopping Table

### Meaning

Defines the frequencies used by the reader when frequency hopping is configured.

The interface states that frequencies are separated by underscores.

### Example format

```text
FREQUENCY1_FREQUENCY2_FREQUENCY3
```

### When to use

Use a custom frequency table only when:

- The regulatory region permits it
- The reader firmware supports it
- The RF environment requires specific frequency planning
- The configuration has been tested

### Recommendation

Use the default regional frequency plan unless there is a clear engineering reason to change it.

---

# 24. Hop Mode

### Meaning

Controls how the reader changes between operating frequencies.

### When to use

Usually leave the default setting unless the RF environment or regulatory requirements require a specific hopping behavior.

### Testing considerations

Check:

- Read rate
- Interference
- Multiple-reader operation
- Compliance with the selected region

---

# 25. Maximum Dwell Time of Antenna

**Visible value:** `0`

### Meaning

Controls how long the reader remains on one antenna before moving to another, if supported.

The interface states that 0 means the device does not fix the resident time of a single antenna.

### When to use

A custom dwell time may be useful when:

- Several antennas are connected
- One antenna needs more reading time
- The application requires controlled antenna sequencing

### Recommendation

For a single antenna, leave the default unless testing indicates a need to change it.

---

# 26. RS232 Serial Parameters

Visible values:

- Modbus address: `2`
- Baud rate: `115200`
- Data bits: `8`
- Stop bits: `1`
- Verification/parity: `No Verification`

## 26.1 Modbus Address

Identifies the serial device when a Modbus-style addressing system is used.

The interface indicates a valid range of **1–255**.

## 26.2 Baud Rate

Defines serial communication speed.

```text
115200 baud
```

Both communicating devices must use the same baud rate.

## 26.3 Data Bits

**Value:** `8`

Defines the number of data bits in each serial character.

## 26.4 Stop Bits

**Value:** `1`

Defines the number of stop bits used to mark the end of a serial character.

## 26.5 Verification / Parity

**Value:** `No Verification`

Means no parity checking is configured.

### Complete serial configuration example

```text
Baud rate: 115200
Data bits: 8
Parity: None
Stop bits: 1
```

This is commonly called:

```text
115200 8N1
```

### Important

The connected controller or Raspberry Pi serial configuration must match the reader.

---

# 27. RS485 Serial Parameters

The RS485 section shows similar parameters:

- Modbus address: `2`
- Baud rate: `115200`
- Data bits: `8`
- Stop bits: `1`
- Verification/parity: `No Verification`

### When to use RS485

RS485 is useful for:

- Longer cable distances
- Multi-drop industrial communication
- PLC and industrial controller integration
- Noisy environments when correctly wired

### Wiring considerations

Verify:

- A/B line labeling
- Signal ground requirements
- Termination resistors
- Cable shielding
- Biasing requirements
- Device addressing

Do not assume that every manufacturer's A/B labels are identical.

---

# 28. Application Initialization Parameters

## 28.1 USB Initial Type

**Visible value:** `HID+CDC`

### Meaning

Defines the initial USB interface mode.

- **HID:** Human Interface Device
- **CDC:** Communication Device Class, commonly used for serial-style communication

### When to use

Select the USB mode based on the software or operating system interface required by the application.

---

## 28.2 Device Type

**Visible value:** `Flexible Unit`

### Meaning

Defines the device operating type or application profile.

### Recommendation

Leave the default unless the vendor instructs you to change it for a specific deployment.

---

## 28.3 Maximum Tag Length

**Visible value:** `62`

The interface states that it controls the maximum EPC + BANK data length in bytes.

### When to use

Increase it only when tags contain longer EPC or additional bank data and the firmware supports the required length.

### Recommendation

Use the minimum length that accommodates your tag data to avoid unnecessary message size.

---

## 28.4 Event Queue Length

**Visible value:** `60`

### Meaning

Defines the number of events that can be queued, according to the firmware's queue implementation.

### When to use

A larger queue may be useful when:

- The network is temporarily unavailable
- Many tag events occur quickly
- The server processes data slowly

### Important

An event queue is not automatically the same as permanent non-volatile storage. Confirm whether the queue survives power loss.

---

# 29. Inventory and Tag Operation Settings

## 29.1 Gen2 Session

**Visible value:** `Session 0`

### Meaning

Controls the Gen2 RFID anti-collision/session behavior.

### When to use

Session selection affects how tags participate in repeated inventory rounds.

### Recommendation

Use the default initially. Change it only after testing tag behavior in the real application.

---

## 29.2 Gen2 Target

**Visible value:** `A`

### Meaning

Defines the target state used during Gen2 inventory.

### When to use

This setting affects which tag population is inventoried during repeated rounds.

### Recommendation

Leave the default unless you have a specific anti-collision or repeated-inventory requirement.

---

## 29.3 QValue

**Visible value:** `Auto`

### Meaning

Controls the Gen2 Q parameter used for tag anti-collision.

Q influences the number of available reply slots in an inventory round.

### Auto mode

The reader adjusts the value automatically according to its algorithm.

### When to use manual Q

Only when:

- The tag population is known
- Auto mode is not performing well
- Controlled RF testing is being performed

---

## 29.4 Profile

**Visible value:** `RF_MODE_7`

### Meaning

Selects an RF operating profile.

The profile can affect modulation, link settings, or other radio parameters depending on firmware.

### Recommendation

Keep the default profile unless the supplier recommends a different one for the tag type or environment.

---

# 30. Inventory Parameters

## 30.1 Inventory Cycle

**Visible value:** `150`

The interface indicates a range of **0–65535 milliseconds**.

### Meaning

Controls the duration or timing of an inventory operation, depending on the firmware definition.

### If the unit is milliseconds

```text
150 ms = 0.15 seconds
```

### When to adjust

- Increase it when more time is needed to detect tags.
- Decrease it when faster cycling is required.
- Test carefully when multiple antennas are used.

---

## 30.2 Inventory Cycle Interval

**Visible value:** `10`

The interface indicates a range of **0–65535 milliseconds**.

### Meaning

Controls the interval between inventory cycles.

### If the unit is milliseconds

```text
10 ms = 0.01 seconds
```

### Recommendation

The combined effect of inventory cycle, interval, processing cycle, and tag filtering should be tested together.

---

## 30.3 Working Antenna

**Visible value:** `1`

### Meaning

Defines which antenna is used for inventory.

The interface indicates that antennas can be separated by commas. If blank, the reader may automatically select antennas.

### Examples

One antenna:

```text
1
```

Multiple antennas:

```text
1,2,3,4
```

### Important for zone detection

If each antenna represents a separate bucket or zone, configure and test antenna selection carefully. The antenna number must be included in the uploaded data.

---

## 30.4 Inventory Mode

**Visible value:** `Pattern 0`

### Meaning

Defines the inventory operating pattern.

### Recommendation

Keep the default until the behavior of each available pattern is documented and tested.

---

# 31. Tag Access Operation Parameters

## 31.1 Operation Timeout Duration

**Visible value:** `1000`

### Meaning

Defines the timeout for tag access operations such as reading or writing tag memory.

The exact unit should be confirmed; it may commonly be milliseconds.

### If milliseconds

```text
1000 ms = 1 second
```

### When to use

Relevant when using:

- Additional Data reads
- Tag memory access
- Tag filtering
- Tag writing or locking

---

## 31.2 Operate Antenna

**Visible value:** `1`

### Meaning

Defines which antenna is used for tag access operations.

### Important

This may be separate from the inventory working antenna setting.

If additional tag memory is read, ensure the selected access antenna can reliably communicate with the target tag.

---

## 31.3 Access Password

### Meaning

Password used for protected tag-access operations.

### Recommendation

Leave blank when no access password is required. If a password is used:

- Store it securely
- Do not expose it in screenshots
- Confirm the tag's access-password configuration
- Test on a non-production tag first

---

# 32. Additional Data, Additional Data 2, and Additional Data 3

These options allow the reader to read additional memory from the tag beyond the basic EPC information.

Each section contains:

- Start Address
- Number of Read Blocks
- Bank

## 32.1 Start Address

Defines the memory address from which reading begins.

## 32.2 Number of Read Blocks

Defines how many memory blocks are read.

## 32.3 Bank

Selects the tag memory bank.

Common Gen2 memory banks include:

| Bank | General purpose |
|---|---|
| Reserved | Passwords and protected control data |
| EPC | EPC and related tag identification data |
| TID | Tag identification information |
| User | User-defined memory |

### When to use

Use Additional Data when the tag contains application information such as:

- Component number
- Batch number
- Product type
- Process ID
- Manufacturing information

### Caution

Incorrect bank, address, or block count settings can cause access failures or unnecessary communication delay. Confirm the tag memory map before enabling this feature.

---

# 33. Tag Filter

### What it means

A tag filter restricts inventory or access operations to tags matching a defined memory pattern.

The interface includes:

- Start Address
- Bank
- FilterMask BIN
- Filter Rule

## 33.1 Start Address

Memory location where the filter comparison begins.

## 33.2 Bank

Memory bank used for the filter.

## 33.3 FilterMask BIN

The binary filter mask used for matching tag data.

## 33.4 Filter Rule

Defines how the filter is applied.

### When to use

Use tag filtering when:

- Only a particular tag family should be read
- A tag contains a known identifier
- Several tag types are present in the same RF field
- You need to exclude unwanted tags

### Recommendation

Test filtering with known-good tags and verify that valid tags are not accidentally excluded.

---

# 34. Time Synchronization

The interface contains a **Time Synchronization** event option.

## Why time synchronization matters

Your application needs reliable timestamps to calculate:

- Tag detection start time
- Tag detection end time
- Approximate visibility duration
- Jig process duration
- Event ordering across multiple readers

## Recommended architecture

```text
Network Time Source
        |
        v
Raspberry Pi system clock
        |
        v
RFID processing software
        |
        v
Database / dashboard
```

### Important distinction

A timestamp may be:

1. Generated by the RFID reader
2. Generated by the server when the message arrives
3. Generated by the Raspberry Pi when it receives the message

These are not necessarily the same time.

### Best practice

Record both where possible:

```text
reader_first_timestamp
reader_last_timestamp
server_received_timestamp
```

This helps identify network delay and reader clock differences.

---

# 35. Recommended Configuration for Initial Industrial Testing

The following is a **starting test configuration**, not a guaranteed final production configuration.

| Parameter | Initial approach |
|---|---|
| Upload protocol | HTTP |
| Hardware interface | Use the actual connected interface |
| Tag Data event | Enabled |
| Heartbeat | Enabled |
| GPI Change | Disabled until a sensor is connected |
| Time Synchronization event | Enable if required by the time-monitoring design |
| First Timestamp | Enabled |
| Last Timestamp | Enabled |
| Antenna Number | Enabled |
| Read Count | Enabled |
| RSSI | Enabled for testing |
| Working antenna | Select the actual antenna |
| QValue | Auto |
| Gen2 Session | Default |
| Antenna power | Start at a moderate tested value |
| DHCP | Use static IP or DHCP reservation |
| Event queue | Keep default initially |
| Additional Data | Disabled until tag memory mapping is confirmed |
| Tag Filter | Disabled until filter requirements are confirmed |
| Automatic restart settings | Keep disabled during initial testing |

---

# 36. Suggested Testing Procedure

## Test 1: Basic tag detection

1. Configure one antenna.
2. Enable Tag Data.
3. Enable Antenna Number, First Timestamp, Last Timestamp, RSSI, and Read Count.
4. Present one tag.
5. Confirm the data reaches the Raspberry Pi.
6. Record the received JSON or message format.

## Test 2: Repeated-read behavior

1. Keep one tag in the antenna field.
2. Observe the read count.
3. Check whether repeated records are sent.
4. Change processing cycle and compare results.
5. Confirm how tag uniqueness works.

## Test 3: Antenna-zone behavior

1. Test one antenna at a time.
2. Confirm the reported antenna number.
3. Place the tag outside the target zone.
4. Check for unwanted reads.
5. Test multiple antennas only after single-antenna behavior is understood.

## Test 4: Timestamp behavior

1. Record the first timestamp.
2. Record the last timestamp.
3. Compare them with the Raspberry Pi receive time.
4. Check whether the reader clock is synchronized.
5. Repeat the test several times.

## Test 5: Network interruption

1. Start the reader and server.
2. Disconnect the network temporarily.
3. Present tags during the interruption.
4. Reconnect the network.
5. Check which records are delivered.
6. Determine whether data is lost, queued, duplicated, or cleaned up.

## Test 6: Power interruption

1. Run the reader under controlled test conditions.
2. Record tag activity.
3. Interrupt power safely.
4. Restore power.
5. Check whether queued data survives the restart.

> Do not assume that an event queue or send buffer survives power loss. Confirm this through testing or vendor documentation.

---

# 37. Important Difference: Buffer vs Permanent Storage

| Component | Typical purpose | Power-loss protection |
|---|---|---|
| Send buffer | Temporarily holds outgoing communication data | Usually not guaranteed |
| Event queue | Holds pending events according to firmware logic | Must be confirmed |
| RAM | Temporary working memory | Usually lost after power loss |
| Flash / MicroSD / database | Can provide persistent storage | Depends on implementation and write strategy |

For a system that must not lose tag data during power failure, use an external data-logging design such as:

```text
SLD1010 / SIM7100
        |
        | Ethernet
        v
Raspberry Pi
        |
        +--> Local persistent queue/database
        |
        +--> HTTP upload to central server
```

The Raspberry Pi application should:

1. Receive the tag message.
2. Validate the message.
3. Save it to local persistent storage.
4. Mark it as pending.
5. Upload it to the server.
6. Mark it as successfully sent only after confirmation.
7. Retry unsent records after network recovery.

---

# 38. Troubleshooting Checklist

## Reader is not uploading data

- Check the upload URL.
- Check the Raspberry Pi IP address.
- Check the port.
- Check the endpoint path.
- Confirm the server is running.
- Confirm the server listens on the correct network interface.
- Check firewall rules.
- Check whether the reader and server are on the same subnet.
- Inspect the HTTP request logs.

## Only one antenna is reporting

- Check Working Antenna.
- Confirm antenna cabling.
- Confirm antenna power.
- Check whether antenna uniqueness is enabled.
- Verify the antenna number in the uploaded data.
- Test antennas individually.

## Tag is detected outside the target zone

- Reduce antenna power gradually.
- Check antenna orientation and placement.
- Check metal reflections.
- Adjust the physical enclosure.
- Test tag orientation.
- Use filtering or uniqueness settings only after understanding their behavior.

## Data is duplicated

- Check Read Count.
- Check Data Processing Cycle.
- Check tag uniqueness settings.
- Check event queue retry behavior.
- Add deduplication logic in the Raspberry Pi application using a tag ID, antenna, and timestamp window.

## Data is lost during network failure

- Check whether the reader supports persistent storage.
- Check event queue behavior.
- Check cleanup settings.
- Implement local persistent storage on the Raspberry Pi.
- Use acknowledgements only when the protocol behavior is understood.

---

# 39. Recommended Data Fields for the Raspberry Pi Database

A practical database record could contain:

```text
id
reader_name
tag_id
antenna_number
rssi
read_count
first_timestamp
last_timestamp
server_received_timestamp
raw_message
upload_status
retry_count
created_at
```

## Why store raw_message?

The raw message helps troubleshoot:

- Format changes
- Missing fields
- Unexpected timestamps
- Duplicate records
- Firmware behavior
- Parsing errors

---

# 40. Final Recommendations

For the first implementation:

1. Configure one antenna and one tag.
2. Confirm the exact uploaded message format.
3. Enable first and last timestamps.
4. Record the antenna number.
5. Use a stable network address for the Raspberry Pi.
6. Build local persistent storage on the Raspberry Pi.
7. Test network and power interruptions.
8. Do not depend only on RSSI to determine exact distance or jig position.
9. Use a physical sensor/GPI signal if exact mechanical start and end positions are required.
10. Keep a configuration backup before production deployment.

---

## Configuration Change Log

| Date | Parameter changed | Old value | New value | Reason | Test result |
|---|---|---|---|---|---|
| YYYY-MM-DD | Example | Default | New value | Test reason | Pass/Fail |

---

## Open Questions to Confirm with the Vendor

The following items should be verified using the official SIM7100/SLD1010 documentation:

- Exact units for Response Timeout Duration
- Exact behavior of Old Data Cleanup Time
- Meaning of Initial Count Condition
- Remote Command Recovery modes
- Antenna power scaling
- Exact behavior of Data Processing Cycle
- Exact behavior of Inventory Cycle and Interval
- Time source and timestamp format
- Whether event queue data survives power loss
- Whether the send buffer is persistent
- HTTP method and acknowledgement requirements
- Exact GPI trigger modes and electrical specifications
- Meaning of each RF profile
- Exact tag uniqueness algorithm
- Supported frequency region for the installation country

