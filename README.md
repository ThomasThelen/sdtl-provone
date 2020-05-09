# sdtl-provone
Examples of SDTL embedded in ProvONE 

### Repository Structure

`single-command/`: Creates a single variable.

`two-commands/`: Creates two variables.

`two-connected-commands/`: Creates a variable which is then used by a second variable.

`three-connected-commands/`: Creates a variable which is used by a second variable. The second variable is used by a third variable.

`two-input/`: Creates two variables and then a third variable that uses uses both of them.

### Overview
The general idea is that provone is first generated and the SDTL is placed in some of the provone objects. The `provone:Program` object holds most of the metadata and is used to represent commands.

### Describing Commands
Commands are described with `provone:Program`. There is generally a top level `provone:Program` that represents the actual script file. The C2Metadata metadata about the file can be placed in this object.

```
{
  "@id": "file://my_script.R",
  "@type": "provone:Program",
  "provone:hasSubProgram": [
    {"@id": "#command_1"},
    {"@id": "#command_2"},
    {"@id": "#command_3"},
  ]
}
```

### Ports
Ports are objects that represent abstract inputs and outputs to programs. When new variables are created, an associated port is created to represent this. Likewise, when a command uses data, a port is used to represent this usage.

```
{
  "@id": "#command_1",
  "@type": "provone:Program",
  "provone:hasOutPort": {"@id": "variable_creation_port"}
}

```


### Connecting Ports
Ports can be connected to show that the output of one or many commands is used as the input in another command. `provone:Channel`s are used to connect the ports. The `provone:Channel` itself doesn't contain much metadata, and there isn't any SDTL embedded in it.


## Notes

1. Not using dcterms:identifer