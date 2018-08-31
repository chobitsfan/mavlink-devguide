# How to Define MAVLink Messages & Enums

MAVLink enums, messages, mission commands, and other elements are [defined within XML files](../messages/README.md) and then converted to libraries for [supported programming languages](../README.md#supported_languages) using a *code generator*.

This topic provides practical guidance for defining and extending MAVLink XML elements, including conventions and best-practice.

> **Note** For detailed information about the file format see [MAVLink XML Schema](../guide/xml_schema.md#messages) (you can also inspect [common.xml](https://github.com/mavlink/mavlink/tree/master/message_definitions/v1.0/common.xml) and other dialect files). 


## Where are the MAVLink XML Files Located?

The "official" project XML files are stored in the Github repo [mavlink/mavlink](https://github.com/mavlink/mavlink/) under [/message_definitions/v1.0/](https://github.com/mavlink/mavlink/tree/master/message_definitions/v1.0/).

MAVLink systems typically fork and maintain a copy of this repo (e.g. [ArduPilot/mavlink](https://github.com/ArduPilot/mavlink)). The downstream repo should pull **common.xml** changes (see next section) down from the main repo and push dialect-specific changes back to it.

> **Tip** The official repo is forked and/or cloned into your environment when you [Install MAVLink](../getting_started/installation.md).

<span></span>
> **Note** A project/dialect doesn't *have to* push changes back to MAVLink. 
  However this makes sense if you want to publish your messages more widely, and potentially get them moved into the **common.xml** message set.


## Where Should MAVLink Elements be Created?

The enums and messages that are generally useful for many flight stacks and ground stations are stored in a file named [common.xml](../messages/common.md), which is managed by the MAVLink project.
The MAVLink elements supported by a particular autopilot system or protocol are referred to as *dialects*. 
The *dialects* are stored in separate XML files, which typically `include` (import) **common.xml** and define just the elements needed for system-specific functionality.

> **Note** When a MAVLink library is generated from a dialect file, code is created for all messages in both the dialect and any included files (e.g. **common.xml**), and entries for a particular enum are merged. 
The generator reports errors if there are name or id clashes between imported messages or enum entries.

Where you define an element depends on whether it is common or a dialect, and whether the project is public or private.

**Elements that are potentially useful for multiple ground stations and autopilots**

- Add these elements to **common.xml** ([mavlink/message_definitions/v1.0/common.xml](https://github.com/mavlink/mavlink/tree/master/message_definitions/v1.0/common.xml)) in the Github **mavlink/mavlink** repo.
- Raise a PR with your suggested changes and discuss with the MAVLink project through that mechanism.

**Elements specific to a particular MAVLink dialect**

- Add these elements to the dialect file in the owning system's fork of the repo.
- Raise a PR with your suggested changes and discuss with the dialect project through that mechanism.
- The dialect project should then (ideally) push the changes back to *mavlink/mavlink*. 

**Elements for a private project**

- If your enums/messages won't ever sync back to the MAVLink project then define them wherever you like!


## Creating a Dialect File

To create a new dialect file:
1. Fork [mavlink/mavlink](https://github.com/mavlink/mavlink/) for your system and clone to your system
1. Create a dialect file named after your MAVLink system (e.g. flight stack) in **message_definitions/v1.0/**
1. Copy the following text into the new file.
   ```xml
   <?xml version="1.0"?>
   <mavlink>

       <include>common.xml</include>
       <!-- <version>9</version> -->
       <dialect>8</dialect>

       <enums>
           <!-- Enums are defined here (optional) -->
       </enums>

       <messages>
           <!-- Messages are defined here (optional) -->
       </messages>
    
   </mavlink>
   ```
   The template assumes that your dialect:
   - imports **common.xml** (`<include>common.xml</include>`)
   - takes its version from **common.xml**  (which is why the `version` tags are commented out).

1. Update the `include`(s):
   - if the dialect is not based on **common.xml** remove the existing `include` line
   - Add additional `<include> </include>` elements to import additional files/dialects.
     > **Note** Includes in nested files are ignored.
1. Update the `version`:
   - Most dialects should leave the version commented out (i.e. all dialects that include **common.xml**).
   - Dialects that are *not* based on **common.xml** can uncomment the `<version>6</version>` line and use whatever version is desired.
       > **Note** The `version` specified in the top level file is used by default, if present. 
         If it is not present in the file, then a `version` from an included file is used. 
1. Update the `<dialect>8</dialect>` line to replace `8` with the next-largest unused dialect number (based on the other files in the folder).
1. Optionally remove the `enums` or `messages` sections if you don't plan on declaring any elements of these types.
1. Add enums or messages as described in the following sections.
1. Save the file, and create a PR to push it back to the **mavlink/mavlink** project repo.


## Messages

[Messages](../guide/xml_schema.md#messages) are used to send data between MAVLink systems (including commands, information and acknowledgments).

Every message has mandatory `id` and `name` attributes.
[Serialised packets](../guide/serialization.md#packet_format) include the `id` in the [message id](../guide/serialization.md#v1_msgid) section and an encoded form of the message data within the [payload](../guide/serialization.md#v1_payload) section. 
The `name` is typically used by generators to name methods for encoding and decoding the specific message type.
When a message is received the MAVLink library extracts the message id to determine the specific message, and uses that to find the appropriately named method for decoding the payload.

A typical message ([SAFETY_SET_ALLOWED_AREA](../messages/common.md#SAFETY_SET_ALLOWED_AREA)) is shown below:

```xml
    <message id="54" name="SAFETY_SET_ALLOWED_AREA">
      <description>Set a safety zone (volume), which is defined by two corners of a cube. This message can be used to tell the MAV which setpoints/waypoints to accept and which to reject. Safety areas are often enforced by national or competition regulations.</description>
      <field type="uint8_t" name="target_system">System ID</field>
      <field type="uint8_t" name="target_component">Component ID</field>
      <field type="uint8_t" name="frame" enum="MAV_FRAME">Coordinate frame. Can be either global, GPS, right-handed with Z axis up or local, right handed, Z axis down.</field>
      <field type="float" name="p1x" units="m">x position 1 / Latitude 1</field>
      <field type="float" name="p1y" units="m">y position 1 / Longitude 1</field>
      <field type="float" name="p1z" units="m">z position 1 / Altitude 1</field>
      <field type="float" name="p2x" units="m">x position 2 / Latitude 2</field>
      <field type="float" name="p2y" units="m">y position 2 / Longitude 2</field>
      <field type="float" name="p2z" units="m">z position 2 / Altitude 2</field>
    </message>
```


### Creating a Message

Messages must be declared between the `<messages></messages>` tags in either **common.xml** or *dialect* files. 
Each message is defined using `<message id="" name="LIBRARY_UNIQUE_NAME"> ... </message>` tags (with unique `id` and `name` attributes).

> **Tip** The only only difference between messages defined in **common.xml** or *dialect* files is they they must use different `id` ranges in order to ensure that the `ids` are unique. See [Message Id Ranges](#message_id_ranges) for more information.

The main rules for messages are:
- Messages **must** include the mandatory `id` and `name`
  - These must be unique across the generated library.
  - See [Message Id Ranges](#message_id_ranges) below for more information.
- Messages *should* (very highly recommended) include a `description`. <!-- update if this becomes mandatory -->
- [Point to point messages](../protocol/overview.md#point_to_point) *must* include a field for `target_system` (exactly as shown above).
- [Point to point messages](../protocol/overview.md#point_to_point) that are relevant to components *must* include a field for `target_component`(exactly as shown above).
- The total payload size (for all fields) must not exceed 255 bytes.
- All other fields are optional.
- There may be no more than 64 fields.
- The `<wip/>` tag may be added to messages that are still being tested.
- Fields:
  - must have unique `name`s within a message.
  - *should* have a description.
  - *should* use the `units` attribute rather than including units in the description. 
    Each field should only have **one** or no units.
  - *should* use the `enum` attribute where possible results are finite/well understood.

> **Warning** You cannot rely on generators to fully test for compliance with the above rules. 
  The *mavgen* code generator tests for duplicate message ids, duplicate field names and messages with more than 64 fields.
  It does not check for other issues (e.g. duplicate names, or over-size payloads). 
  Other generators may provide better validation


#### Message Id Ranges {#message_id_ranges}

All messages within a particular generated library must have a unique ID - this is important because the `id` is used to determine the format of the message payload (i.e. what generated method can decode the message).

For MAVLink 2, each dialect is allocated a specific range from which an id can be selected. 
This ensures that any dialect can include any other dialect (or common.xml) without clashes.
It also means that messages can move from a dialect to common.xml without any code needing to change.

When creating a new message you should select the next unused id for your dialect (after the last one defined in your target dialect file).

The allocated ranges are listed below.

Dialect | Range
--- | ---
Common.xml | 300 - 10000
uAvionix.xml | 10001-10999
ArduPilotMega.xml | 11000 - 11999
icarous.xml | 42000 - 42999

> **Tip** If you are creating a new public dialect, [create an issue](https://github.com/mavlink/mavlink/issues/new) to request your own message id range. For private dialects, you can use whatever range you like.

You should not create messages with ids in the "MAVLink 1" range (MAVLink v1 only has 8 bit message IDs, and hence can only support messages with ids 0 - 255). 
<!-- Note, historically ids 150 to 230 were reserved for dialects. People should not be creating messages in this range, so I'm not going to explain that-->

### Modifying a Message

Changing the name or id of a message will make it incompatible with older versions of the generated library. 

Adding or removing a field, or changing the name or type of a field, will make a message incompatible with older versions of the generated library (the generated message decoding method is hard coded with the field number, [order](../guide/serialization.md#crc_extra), type and position at build time - if these change, decoding will fail).

If a message needs to be changed in these ways then there are several options:
* A new message can be created with the desired behaviour. 
  At some point the old message may be marked as [deprecated](../guide/xml_schema.md#deprecated).
* The message can be updated, and the dialect version number iterated. 

For either case, all users of the message will need to be updated with new client libraries.

For a message in **common.xml** either change requires the agreement of major stakeholders 
- Create a PR and discuss in the MAVLink developer meeting.

> **Tip** Before proposing changes to **common.xml** check the codebase of major stakeholder to confirm impact.

It is possible to change the message and field descriptions without breaking binary compatibility.
Care should still be taken to ensure that any changes that alter the way that the field is interpreted are agreed by stakeholders, and handled with proper version control.

Messages are very rarely deleted, as this may break compatibility with legacy MAVLink 1 hardware that is unlikely to be updated to more recent versions.


#### MAVLink 2 Message Extensions {#message_extensions}

The exception to the above rul is that when using [MAVLink 2](../guide/mavlink_2.md) you can add new fields to messages with an `id` of 0 - 255 using the [<extensions>](../guide/mavlink_2.md#message_extensions) tag *without breaking compatibility*. 
These fields will not be sent when the MAVLink 1 protocol is used.

Otherwise the rules are the same; once added you cannot modify or remove fields. 
You can however continue to add new fields as long as you do not exceed the maximum field number or payload size limits.

> **Note** Extension fields are appended to the end of the original MAVLink 1 message fields and are not reordered. 
  A decoding library that does not understand a new extension field will ignore it, but will not incorrectly decode the fields that it does understand.


<!-- A FEW NOTES

common.xml
- MAVLink 1: 0-93, 100-149, 230-235,241, 254
- MAVLink 2 - 256-270, 299-300, 310, 311, 320-324, 330-333
APM
- MAVLink 1: 150 - 219, 226
- MAVLink 2: ardupilot specific mavlink2 messages starting at 11000 - 11000->11032 (not filled)
uAvionix.xml
- MAVLink 2: 10001
icarous.xml
- MAVLink 2: 42000
ASLUAV
-MAVLink 1:  78, 79, 201-212
MatrixPilot
- MAVLink 1: 150-158, 170-188

Open questions:
- What other rules/guidance can we give. Some ideas:
  - GCS find it easier to index /parse arrays, so consider using arrays for fields that represent sequential items - e.g. port values.
  - Future proof - don't assume that values will be static forever.
  - ?

-->



## Enums {#enums}

[Enums](../guide/xml_schema.md#enum) are used to define named values that may be used as options in messages - for example to represent errors, states, or modes.

Every enum has mandatory `name` attribute and may contain a number of `entry` elements (with enum-unique names) for the supported values. 
The *same* `enum` may be declared in **common.xml** and multiple dialects.
The generated library will merge the entry values, and should report an error if there are any duplicate names.

A typical enum ([LANDING_TARGET_TYPE](../messages/common.md#LANDING_TARGET_TYPE)) is shown below:

```xml
<enum name="LANDING_TARGET_TYPE">
    <description>Type of landing target</description>
    <entry value="0" name="LANDING_TARGET_TYPE_LIGHT_BEACON">
        <description>Landing target signaled by light beacon (ex: IR-LOCK)</description>
    </entry>
    <entry value="1" name="LANDING_TARGET_TYPE_RADIO_BEACON">
        <description>Landing target signaled by radio beacon (ex: ILS, NDB)</description>
    </entry>
    <entry value="2" name="LANDING_TARGET_TYPE_VISION_FIDUCIAL">
        <description>Landing target represented by a fiducial marker (ex: ARTag)</description>
    </entry>
    <entry value="3" name="LANDING_TARGET_TYPE_VISION_OTHER">
        <description>Landing target represented by a pre-defined visual shape/feature (ex: X-marker, H-marker, square)</description>
    </entry>
```


### Creating an Enum

Enums must be declared between the `<enums></enums>` tags in **common.xml** and/or *dialect* files.
Each enum is defined using `<enum name="SOME_NAME"> ... </enum>` tags (with a `name` attribute).

> **Tip** There is no difference between enums defined in **common.xml** or *dialect* files (other than management of the namespace). 
  
The main rules for enums are:
- Enums **must** include the mandatory `name` attribute.
  - Entries are merged for all enums that share the same `name`.
- Enums *should* (very highly recommended) include a `description`. <!-- update if this becomes mandatory -->
  If enums are merged, only one description will be used (usually the first that is encountered).
- Enums *may* be marked as deprecated.
- Enums *must* have at least one enum entry.
- Entries:
  - *must* have a `name` attribute.
    - The `name` must be unique across all entries in the enum.
    - By *convention*, the `name` should be prefixed with the enum name (e.g. enum `LANDING_TARGET_TYPE` has entry `LANDING_TARGET_TYPE_LIGHT_BEACON`).
  - *may* have a `value` attribute, and if assigned this must be unique within the (merged) enum.
    A value will be automatically created for the generated library if not assigned.
  - *should* (very highly recommended) include a `description` element. 
  - may represent bitmasks, in which case values will increase by a power of 2.
  - *may* be marked as deprecated.

> **Warning** You cannot rely on specific generators to fully test for compliance with the above rules.
  *mavgen* tests for duplicate names in enums, duplicate names for (merged) enum entries, duplicate values for enum entries.

  
### Modifying an Enum

Changing the name or removing an enum *will* make any messages that use the enum incompatible with older versions of the generated library.
Similarly, changing an enum entry `name` or `value`, or removing an enum entry, *will* make messages that use the enum incompatible with older versions of the generated library.

Care must be taken when adding a new enum entry/value as this *may* make the generated library incompatible:
- Autogenerated entry values may change
- Client code may not handle new values.

If an enum needs to be changed then there are several options:
* A new enum can be created with the desired entries. 
  At some point the old enum may be marked as [deprecated](../guide/xml_schema.md#deprecated).
* The enum can be updated, and the dialect version number iterated. 

For either case, all users of the enum will need to be updated with new generated libraries.

> **Tip** Before proposing changes to **common.xml** check the codebase of major stakeholder to confirm impact.

For an enum in **common.xml** either change requires the agreement of major stakeholders
- Create a PR and discuss in the MAVLink developer meeting.

It is possible to change enum/enum entry descriptions without breaking binary compatibility.
Care should still be taken to ensure that any changes that alter the way that they are interpreted are agreed by stakeholders, and handled with proper version control.

Enums are very rarely deleted, as this may break compatibility with legacy MAVLink 1 hardware that is unlikely to be updated to more recent versions.


## Mission Commands {#mission_commands}

TBD

<!-- 

Mission commands are used to define operations used in autonomous missions.
They are defined as entries in the [MAV_CMD](../messages/common.md#MAV_CMD) enum and are encoded into [MISSION_ITEM](../messages/common.md#MISSION_ITEM) or [MISSION_ITEM_INT](../messages/common.md#MISSION_ITEM_INT) messages when a mission is sent  (see [Mission Protocol](../protocol/mission.md)).

> **Tip** The [Command Protocol](../protocol/command.md) can be used to send these commands outside of missions, encoded in a [COMMAND_LONG](../messages/common.md#COMMAND_LONG) or [COMMAND_INT](../messages/common.md#COMMAND_INT) message. 
  Using an existing mission command is usually better/easier than creating a new separate message for use outside of missions.
  

Assumptions/Questions
- ...

```xml

    <message id="76" name="COMMAND_LONG">
      <description>Send a command with up to seven parameters to the MAV</description>
      <field type="uint8_t" name="target_system">System which should execute the command</field>
      <field type="uint8_t" name="target_component">Component which should execute the command, 0 for all components</field>
      <field type="uint16_t" name="command" enum="MAV_CMD">Command ID (of command to send).</field>
      <field type="uint8_t" name="confirmation">0: First transmission of this command. 1-255: Confirmation transmissions (e.g. for kill command)</field>
      <field type="float" name="param1">Parameter 1 (for the specific command).</field>
      <field type="float" name="param2">Parameter 2 (for the specific command).</field>
      <field type="float" name="param3">Parameter 3 (for the specific command).</field>
      <field type="float" name="param4">Parameter 4 (for the specific command).</field>
      <field type="float" name="param5">Parameter 5 (for the specific command).</field>
      <field type="float" name="param6">Parameter 6 (for the specific command).</field>
      <field type="float" name="param7">Parameter 7 (for the specific command).</field>
    </message>
```
    
    
    Need to cover creating, modifying, deleting, adding to messages.
-->