# SlvCtrl+ protocol 1.0 (DRAFT!)

:warning: This protocol version is not yet implemented/supported!

## General
### Command separation
The commands and their responses always MUST be separated by a new line (`\n`).


### Parameter separation
A command's parameters MUST be separated by a space:
```
set-flow 50 100
```

## Commands
### Command `introduce`
This is the first command the server sends to the component once it established a successful serial connection.
The component MUST answer this command with some basic information about itself. (See *Response* section)

#### Request
No parameters

#### Response
```
introduce;{device type: string},{firmware version: int},{protocol version: int}
```

Example: 
```
introduce;air_valve,10223,10000
```

The number `10223` MUST be read as `1.2.23`.
The number `10000` MUST be read as `1.0.0`.

### Command `attributes`
Returns a list of attributes this component offers for read and/or write.

The attribute names MUST only contain lower case letters a-z, numbers and dashes and MUST have at least the length
of 1 (`^[a-z0-9-]+$`).

### Request
No parameters

### Response
```
attributes[;{attribute 1}:{{attribute type}}[{data type}]},...]
```

Example:
```
attributes;flow:rw[0-100],pressure:ro[10-20]
```

#### Attribute types
| Type   | Description |
|--------|-------------|
| `ro`   | read-only, attribute can only be read |
| `wo`   | write-only, attribute can only be written |
| `rw`   | read-write, attribute can be read and written |

#### Data types
| Type   | Description | Example |
|--------|-------------|---------|
| String  | Any string  | `str` |
| Integer | An integer | `int` |
| Float   | A float | `float` |
| Boolean | A boolean | `bool` |
| List    | A list of options (of any other data type) | <code>foo&#124;bar&#124;baz</code> or <code>8&#124;16&#124;32&#124;64</code> |
| Range   | A range | `0-100` (of either integer or float) |

### Command `status`
Returns the current status of the component. It MUST return a comma separated list with all attributes and their values.
If a value cannot be determined at the moment it MUST be omitted.

#### Request
No parameters

#### Response
```
status[;{attribute name 1}:{value 1},{attribute name 2}:{value 2},...]
```

Example:
```
status;flow:55
```

Example in case value cannot be determined at the moment:
```
status;flow:
```

### Attribute commands `get-...` and `set-...`
The component MUST implement a command to get a value and/or write a value for each attribute.

The command names MUST either start with `get-` or `set-` depending on whether they read or write the attribute.

Parameters MUST be provided as a space separated list: `set-flow 20`.

The response of a set command MUST always contain the command name plus all its parameters with an additional 
property/value list which has to at least contain a `status` about the set command: `set-flow;20;status:successful`.

#### Possible states
| State  | Description |
|--------|-------------|
| `successful` | Value was set successfully |
| `failed`     | Setting the value failed |
| `unknown`    | Currently it's unknown whether setting the value was successful or not |

If state is `failed` or `unknown` an additional `reason` field MAY bet set with more detailed info (error code, etc).

#### Response
The response of a get command MUST return the command name and the current value. If there are multiple values they
MUST be separated by a comma. If the value cannot be determined the value list MUST be empty.

Example in case of success:
```
get-flow;50;status:success
```

Example if value cannot be determined at the moment:
```
get-flow;;status:unknown,reason:in_motion
```

## Complete example
* `-->` describes input the server sends to the component
* `<--` describes output the component returns as response to a command sent by the server to the component

```
--> introduce\n
<-- introduce;air_valve,10223,1\n
--> attributes
<-- attributes;flow:rw[0-100],pressure:ro[10-20]
--> status\n
<-- status;flow:100\n
--> set-flow 50\n
<-- set-flow;300;status:failed,reason:value_out_of_range\n
--> set-flow 50\n
<-- set-flow;50;status:successful\n
--> get-flow\n
<-- get-flow;50\n
```