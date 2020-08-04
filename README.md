# sdtl-provone


## Repository Structure

`examples/`: Directory holding examples of a ProvONE data model paired
with SDTL 

`images/`: Images found in this README

## Overview

The broad goal is to take SDTL metadata and create an RDF data model
that describes the inner-workings of the script that the SDTL describes.
The ProvONE data model has a sufficient vocabulary to represent the
creation and flow of information in the script; SDTL can then be
embedded in the provone/prov objects to give a much more rich data
model.


## Mapping
The creation of the RDF model and embedding of SDTL properties can be
automated. This section is concerned with the specifics of how that
mapping should happen.

### Notation
When generating the model, the identifers should look similar between
generations (ie not random identifiers). The following section outlines
a naming convention for identifiers and labels.

#### Assigning Identifiers

The naming convention for identifiers was taken from the `Patterned
URIs` section in
[Linked Data Patterns](https://patterns.dataincubator.org/book/).

Because there are many different classes between prov, ProvONE, and SDTL
it's impractical to create a mapping to plurized forms.

The general form of the identifier is...

`#lowercase_type/current_type_count`. where `current_type_count` is the
count of the particular object.

Example:

    If a document has three provone:Program objects and one provone:Port, the RDF model should have three identifiers that resemble
    1. #program/1
    2. #program/2
    3. #program/3
    4. #port/1
    
Note that there aren't any nested URIs and they are referenced from the
top level (not `#program/1/port/1`); instead, accessing the first port
should be `#port/1`.


###### Prospective
1. provone:Workflow -> workflow
2. provone:Program -> program
3. provone: Port -> port
4. provone:Channel -> channel

###### Retrospective
1. provone:Execution -> execution
2. prov:Entity -> entity
3. provone:Data -> data
4. prov:Usage -> usage
5. provone:Document -> document
6. provone:Visualization -> visualization
7. prov:Generation -> generation
8. prov:Association -> association
9. provone:Collection -> collection 


###### SDTL
SDTL has many more classes than Prov & ProvONE; so follow the general
case of lower-caps and counting the number of instances the class has.

1. allTextVariablesExpression -> alltextvariablesexpression
2. functionArgument -> functionargument
3. command -> command


#### Labels
Every object _should_ have an `rdfs:label`. This label should give some
sort of description of the object's identifier. This is used/read by
someone querying the data model and is important for readability.

Because the RDF objects are programmatically generated, the labels will
have to be generated algorithmically. This leaves the challenge of
encoding enough information in the label in an automated way such that
it's easier to understand and more user friendly than the identifier.

The ProvONE ontology has an `rdfs:label` defined for each class however,
it's not in the context of this application.

Note that labels are _not_ unique across objects.

The general form for a label is similar to the identifier,

`#type current_type_count`

The main difference is the replacement of `/` with a space. The
requirement that the type names are lower-cased is relaxed.

It would be nice to give names that have more context based on som
heuristics.

For example, if there's a `provone:Program` that represents a script (ie
is not the predicate of a `provone:hasSubProgram`) a descriptive label
could be

`rdfs:label`: Top level script {script-name}

###### Examples

**Workflow** 

`rdfs:label`: `Workflow current_type_count`

**Program**:
 
`rdfs:label`: `Program current_type_count`
 
**fileName**

`rdfs:label`: `fileName current_type_count`

##### SDTL

SDTL already has descriptions for major SDTL objects in the schema.
These can be used to generate the label for each SDTL object.

    Example:
    {
        "@id": "#filename/1,
        "@type": "sdtl:fileName",
        "rdfs:label": "fileName 1",
    }

```
{
    "@id": "#workflows/1",
    "@type": "provone:Workflow",
    "rdfs:label": "Workflow 1",
    "provone:hasSubProgram": [
        { "@id": "#programs/1" },
        { "@id": "#programs/2" }
    ]
},

{
"@id": "#executions/1",
"@type": "provone:Execution",
"rdfs:type": "Execution 1"
}

{
    "@id": "#programs/1",
    "@type": "provone:Program",
    "rdfs:label" "Program 1",
    "provone:wasPartOf: "#executions/1"
}

{
    "@id": "#programs/2",
    "@type": "provone:Program",
    "rdfs:label" "Program 1",
    "provone:wasPartOf: "#executions/1"
}

```



### Constructing the provenance model
 
Creating the provenance model from the SDTL happens in three steps:
1. Create the retrospective model
2. Create the prospective model
3. Connect the retrospective and prospective models together

#### Constructing the Retrospective Provenance
The retrospective provenance is concerned with the following RDF 
objects:
1. provone:Execution
2. prov:Entity
3. prov:Generation
4. prov:Usage
5. prov:Association


The execution of a script is represented with a `provone:Execution`. The
`provone:Execution` _can_ have inputs and outputs however, the top level
`provone:Execution` in this case doesn't. The execution of each command
in the script is also represented by the `provone:Execution`. Also note
that there isn't an object that holds the top level
script-representations, as opposed to the `prov:Workflow` that
prospective provenance has.

Unless noted, referencing `provone:Execution` refers to the _command_
level `provone:Execution`.
 
For each script file:
1. Create a `provone:Execution` to represent the execution of the
   script. This is referred to as the script-level execution. 
   
##### Alogrithm

For each SDTL Command, 
1. Create a new `provone:Execution` 
2. Connect it to the script-level `provone:Execution` with
   `provone:wasPartOf`
3. Create a new `prov:Association` and connect the `provone:Execution`
   to it with `prov:qualifiedAssociation`
   
For each created variable: 
1. Create a `prov:Entity`
2. Connect the `prov:Entity` to the `provone:Execution` with
   `prov:wasGeneratedBy`
3. Create a `prov:Generation`
4. Connect the `prov:Execution` to the `prov:Generation` with
   `prov:qualifiedGeneration`
5. Connect the `prov:Generation` to the `prov:Entity` with
   `prov:hadEntity`


For each used variable:
1. Create or use an existing `prov:Entity` that represents the used
   variable
2. Connect the `provone:Execution` to the entity with `prov:Used`
3. Create a prov:Usage and connect the `prov:Execution` to it with
   `prov:qualifiedUsage`


 
The relationship between SDTL command executions and the parent
execution representing the script's execution.

![](./images/parent-execution.svg)


#### Constructing the Prospective Provenance
The prospective provenance is concerned with the following objects:
1. provone:Program
2. provone:Port
3. provone:Workflow

##### Algorithm
 
For each script,
1. Create a top level `provone:Program` that represents the script-level
   program
2. Create a `provone:Workflow` object if one doesn't already exist. This
   represents a collection of scripts
3. Connect the `provone:Workflow` to the new `provone:program` with
   `provone:hasSubProgram`
 
For each command,
1. Create a `provone:Program`
2. Connect the top level `provone:Program` to it with
   `provone:hasSubProgram`

If the `provone:Program` creates a variable:
1. Create a `provone:Port`
2. Connect the `provone:Program` to it with `provone:hasOutPort`

If the `provone:Program` uses a variable: 

Note that there should be a corresponding `provone:Program` that created
the variable being used. That `provone:Program` _should_ have a
`provone:Port` connected to a `provone:Channel`

1. Create a `provone:Port`
2. Connect it to the target `provone:Channel` with `provone:connectsTo`
3. Connect the `provone:Program` to the `provone:Port` with
   `provone:hasInPort`



#### Connecting Retrospective & Prospective
The retrospective and prospective models are connected at the following
objects:
1. `provone:Program` <-> `prov:Association`
2. `prov:Port` <-> `prov:Generation` & `prov:Usage`
 
##### Algorithm

For each `provone:Execution`/`provone:Program`:
1. Connect the `prov:Association` to the `provone:Program` with
   `prov:hadPlan`

For each created variable:
1. A `prov:Entity` and `provone:Port` should already exist representing
   the new variable
2. A `prov:Generation` should already exist
3. Connect the `prov:Generation` to the `provone:Port` with
   `provone:hadOutPort`

For each used variable:
1. Connect the `provone:Execution` to the used `prov:Entity` with
   `prov:Used`
2. Connect the `prov:Usage` to the `provone:Port` with
   `provone:hadInPort`
   
#### Emdedding SDTL


##### Script Level Metadata
C2Metadata provides output about the script passed to its parser. A sample of this looks like
```
  "id": "program-1",
  "sourceFileName": "",
  "sourceLanguage": "spss",
  "scriptMD5": "518001a968c359366bf7ceb12bf209ea",
  "scriptSHA1": "3dead21a7b31e1409d2ab364cbf4f734366186ad",
  "sourceFileLastUpdate": "2020-04-14T18:38:10+00:00",
  "sourceFileSize": 19,
  "lineCount": 1,
  "commandCount": 1,
  "parser": "spss-to-sdtl",
  "parserVersion": "0.9 Development",
  "modelVersion": "0.9 Development",
  "modelCreatedTime": "2020-04-14T18:38:10+00:00",
```

This metadata is most closely associated with the top level
`provone:Program` and `provone:Exection`.


###### Discarded Terms
Some of these (listed below) don't belong in the a provenance trace.

  `id`: This corresponds to the parser run ID. The provone:Execution or
  provone:Program will already have a unique ID.

  The others include `parser`, `parserVersion`, `modelVersion`,
  `modelCreatedTime`

###### Kept Terms
The rest of the terms,

  `sourceFileName`: Name of the file (Kept because @id does _not_ need to be the filename)

  `sourceLanguage`: Language of the script
  
  `scriptMD5`: Hash of the script
  
  `scriptSHA1`: Hash of the script
  
  `sourceFileLastUpdate`: The last time the file was modified
  
  `sourceFileSize`: The size of the file
  
  `lineCount`: The number of source lines in the program
  
  `commandCount`: The number of SDTL commands inside


###### Embedding
This metadata should be placed in the script-level `provone:Execution`
and `provone:Program` objects.
 
 ![](./images/prov.svg)

```json
      {
          "@id": "#program/1",
          "@type": "provone:Program",
          "provone:hasSubProgram": {"@id": "#program/2"},
          "rdfs:label": "Program 1",
          
          "sdtl:sourceFileName": "",
          "sdtl:sourceLanguage": "r",
          "sdtl:scriptMD5": "518001a968c359366bf7ceb12bf209ea",
          "sdtl:scriptSHA1": "3dead21a7b31e1409d2ab364cbf4f734366186ad",
          "sdtl:sourceFileLastUpdate": "2020-04-14T18:38:10+00:00",
          "sdtl:sourceFileSize": 19,
          "sdtl:lineCount": 1,
          "sdtl:commandCount": 1
      }
```


##### Embedding Commands
Command level metadata is modeled inside  `provone:Program` and
`provone:Execution` objects.

An example of a software program with three commands inside would, from
a prospective perspective, look like

```
{
  "@id": "#program/1",
  "@type": "provone:Program",
  "provone:hasSubProgram": [
    {"@id": "#program/2"},
    {"@id": "#program/3"},
    {"@id": "#program/4"},
  ],
  
  "sdtl:sourceFileName": "",
  "sdtl:sourceLanguage": "r",
  "sdtl:scriptMD5": "518001a968c359366bf7ceb12bf209ea",
  "sdtl:scriptSHA1": "3dead21a7b31e1409d2ab364cbf4f734366186ad",
  "sdtl:sourceFileLastUpdate": "2020-04-14T18:38:10+00:00",
  "sdtl:sourceFileSize": 19,
  "sdtl:lineCount": 1,
  "sdtl:commandCount": 1
}
```

Note that `#my_script.R` is the same level `provone:Program` as the one
described in the File Level Metadata section and contains the
script-level SDTL.

![](./images/commands.svg)

A `provone:Program` with SDTL command level metadata looks like
```
{
  "@id": "#program/1",
  "@type": "provone:Program",
  "$type": "Compute",
  "command": "compute",
  "sourceInformation": {
    "@id": "#sourceinformation/1",
    "lineNumberStart": 1,
    "lineNumberEnd": 1,
    "sourceStartIndex": 1,
    "sourceStopIndex": 19,
    "originalSourceText": "compute newVar = 0."
  },

  "variable": {
    "@type": "VariableSymbolExpression",
    "variableName": "newVar"
  },
  "expression": {
    "$type": "NumericConstantExpression",
    "value": "0",
    "numericType": "int"
  }
}
```

It contains information about where in the script the command is (useful since provenance does not always preserve order), what type of command (in this case `sdtl:compute`), and information about the variables used.

Note that I've specified an ID for child objects. In this case, I've
chosen <key_parentKey> as the convention. This was done to avoid blank
nodes (hard to reproduce queries).

##### Embedding Variable Information

Ports are the prospective objects that represent inputs and outputs to
programs. When new variables are created, an associated port is created
to represent this. Likewise, when a command uses data, a port is used to
represent this usage.

A basic provone:Port has the format
```
{
  "@id": "#port/1",
  "@type": "provone:Port"
}
```

![](./images/ports.svg)

The C2Metadata SDTL parser provides information about output from
commands in the `variable` and `expression` properties.

A port object containing this information looks like
```
{
  "@id": "#command_1_outport",
  "@type": "provone:Port"
  "variable": {
    "@id": "#variable_command_1_outport",
    "@type": "VariableSymbolExpression",
    "variableName": "newVar"
  },
  "expression": {
    "@id": "#expression_command_1_outport,
    "@type": "NumericConstantExpression",
    "value": "0",
    "numericType": "int"
    }
}
```


For retrospective provenance, `prov:Entity` objects hold the variable
information. 

```
{
  "@id": "#execution_1_output_entity",
  "@type": "prov:Entity"
  "variable": {
    "@id": "#variable_execution_1_output_entity",
    "@type": "VariableSymbolExpression",
    "variableName": "newVar"
  },
  "expression": {
    "@id": "#expression_execution_1_output_entity",
    "@type": "NumericConstantExpression",
    "value": "0",
    "numericType": "int"
    }
}
```

##### Algorithm

For each

**Top Level SDTL Record** 

1. Discard the `id`, `parser`, `parserVersion`, `modelVersion`, and
`modelCreatedTime` fields
2. Plce the rest of the SDTL in the corresponding, script-level
   `provone:Program`

**Element in the 'command' Array**

1. Place the entire command SDTL object inside the corresponding
   `provone:Program` object.
2. If the command has an `sdtl:expression` or `sdtl:variable`, include
   it in the corresponding `provone:port` if one exists.

For each SDTL object
1. Prepend `sdtl:` to the name (namespace SDTL)

For each `$type` record
1. Remove `$`
2. Prepend `sdtl:` (as with all other SDTL objects)







### Connecting Programs to Ports
Programs need to be linked to related its related provone:Port(s). This can be done two ways:

1. Using provone:hasOutPort to show output
2. Using provone:hasInPort to show usage

An example of a provone:Program that produces the provone:Port above:
```
{
  "@id": "#command_1",
  "@type": "provone:Program",
  "provone:hasOutPort": {"@id": "#command_1_outport"}
}
```

A `provone:Program` can also have multiple inports and outports. Not that the `provone:hasInPort` is using (as the naming implies) the outPort of something else.

```
{
  "@id": "#complex_command",
  "@type": "provone:Program",
  "provone:hasInPort": [
    {"@id": "#command_n1_outport"},
    {"@id": "#command_n2_outport"},
    {"@id": "#command_n31_outport"}
  ],
  "provone:hasOutPort": {"@id": "#complex_outport"}
}
```
Note that this is the same image from the section above. It's included here to review after reading this section

![](./images/ports.svg)


# Misc

## Describing Workflows
As seen in the previous sections, ProvONE allows the grouping of `provone:Program` by with `provone:hasSubProgram`. We previously used it to describe the relationship between the individual commands in the script and the script itself.

In this section, multiple scripts are grouped together by using a specialization of `provone:Program`: the `provone:Workflow`. From the ProvONE specification,

The following diagram describes the workflow:

1. Execute `clean_data.R` that reads a file in and produces output
1. Execute `analyzed_clean_data.R` that uses the output from step 1 to produce an output.
1. Execute `format_analysis.R` that uses the output of step 2 to create another output.

![](./images/workflow.svg)


### Connecting Ports to Ports
Ports can be connected to show that the output of one or many commands is used as the input in another command. `provone:Channel` objects are used to connect the ports to each other. The `provone:Channel` itself doesn't contain much metadata, and there isn't any SDTL embedded in it.

The provone:Channel doesn't contain any SDTL, and the connections are from the related `provone:Port` objects.
```
{
  "@id": "#channel_port1_port2",
  "@type": "provone:Channel"
}
```

The first port would show a connection with
```
{
  "@id": "#port_1",
  "@type": "provone:Port",
  "provone:connectsTo": { "@id": "#channel_port1_port2" }
}
```

The second as
```
{
  "@id": "#port_2",
  "@type": "provone:Port",
  "provone:connectsTo": { "@id": "#channel_port1_port2" }
}
```


![](./images/channel.svg)
