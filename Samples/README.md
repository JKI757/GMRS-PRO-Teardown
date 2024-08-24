The GMRS-PRO has the capability to send binary data (specifically texts and positioning information) between handsets.
While the GMRS-PRO has the ability to receive APRS (though I've not personally tested this), it does NOT transmit APRS. 
Instead it uses a proprietary format. In the documentation for the UV-PRO (the HAM "big brother" to the GMRS-PRO), they
document this "BSS" mode as an alternative to APRS for unlicensed operators. Whether this "BSS" mode is the same as
what's transmitted on the GMRS-PRO is unknown.

...if someone wants to send me a UV-PRO, I'll be happy to test that, though... :)

# Text Messages

The simplest of these messages is a "broadcast text" that looks like this, with the following fields:

<0x01><0x03> BB<0x01>!<0x15>$ABCDEFGHIJKLMNOPQRSS
    ^     ^   ^    ^ ^    ^                     ^____ The message. The $ prefix seems to indicate "message follows"
    |     |   |    | |    +__________________________ The length of the message (in bytes) including the prefix
    |     |   |    | +_______________________________ The ID of the recipient of the message of the message prefixed by !. 
    |     |   |    |                                  If ID is blank, recipient is "everyone"
    |     |   |    +_________________________________ The length of the recipient (in bytes) including the prefix
    |     |   +______________________________________ The ID of the sender, prefixed with a space. 
    |     +__________________________________________ The length of the sender (in bytes), including the prefix
    +________________________________________________ The preamble

A directed message is similar, except the recipient is not blank.

# Location Messages

Location messages are similar to text messages, except they lack prefix. I'm not sure if a text would work if sent
directly to a specific user. I have not tested this.

The format of the location messages are as follows:

<0x01><0x03> BB<0x0d><0x25><0x12><0x25><0x00><0xcf><0xdc><0x00><0x06><0x1f><0x00><0x00><0x00><0x01>
    ^     ^   ^    ^     ^     |           |     |           |     ^     ^     ^     ^     ^    ^^__ Heading (in degrees)
    |     |   |    |     |     |           |     |           |     |     |     |     |     +________ Heading?
    |     |   |    |     |     |           |     |           |     |     |     |     +______________ Speed (in KM/h)
    |     |   |    |     |     |           |     |           |     |     |     +____________________ Speed?
    |     |   |    |     |     |           |     |           |     +_____|__________________________ Altitude in meters 
    |     |   |    |     |     |           |     +___________|______________________________________ Latitude
    |     |   |    |     |     +___________|________________________________________________________ Longitude
    |     |   |    |     +__________________________________________________________________________ Unknown. Doesn't update if != 0x25
    |     |   |    +________________________________________________________________________________ Message Length
    |     |   +_____________________________________________________________________________________ Sender, prefixed with a space
    |     +_________________________________________________________________________________________ Length of sender, including prefix
    +_______________________________________________________________________________________________ One, always one.

## Latitude / Longitude

Seems to be a 24bit signed int scaled to represent 0.12 arcseconds.
This was a pain to figure out. Why they didn't go a different route and just multiply the DD coord is beyond me, as I
believe they could have gotten better precision.

Python code to extract:
```
(int.from_bytes([0x12,0x5f,0x00], byteorder='big', signed=True) * 0.12) / 3600
```

## Altitude

Altitude is packed as a 16bit signed float.
```
(struct.unpack('>h', b'\x06\x1f'))
```

## Speed

If I set the byte identified above to 0xFF, speed is shown as 255km/h on the HT. The following byte might be the MSB 
of a 16bit int (it would make sense). I don't remember testing that, though. 

## Heading

Similar with speed, I know the identified field changes the heading. But there's 360 degrees in a heading, so it'd
likely require two bits. I don't remember testing that either but seems reasonable.

# Misc

The "Call", "Check" and "Nearby People" menu options don't seem to burst long enough to trigger 
Direwolf's decoding. If I get more time I'll fiddle and try to figure it out. If someone wants to post 
some binary/hex dumps of these, that'd be helpful. 
