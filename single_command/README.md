# SDTL With ProvONE

## Background
One deliverable of Whole Tale is to include the output metadata from C2Metadata in a DataONE resource map. The DataONE resource maps are RDF/XML representations of data objects and their relations to each other (via prov and provone).

The ouput we're interested in from C2Metadata is SDTL, which can be serialized in a number of different ways including rdf/xml.

You can find the SDTL schemas [on the C2Metadata SDTL GitLab page](http://c2metadata.gitlab.io/sdtl-docs/)

This document uses a single-command-SPSS-script as a motivating example to demonstrate that the meshing is possible.

The script has a single command which creates a new variable with the initial value 0.

`compute newVar = 0.`

It has the following SDTL output, as JSON-LD

```json=
{
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
  "commands": [
    {
      "$type": "Compute",
      "command": "compute",
      "sourceInformation": {
        "lineNumberStart": 1,
        "lineNumberEnd": 1,
        "sourceStartIndex": 1,
        "sourceStopIndex": 19,
        "originalSourceText": "compute newVar = 0."
      },
      "variable": {
        "$type": "VariableSymbolExpression",
        "variableName": "newVar"
      },
      "expression": {
        "$type": "NumericConstantExpression",
        "value": "0",
        "numericType": "int"
      }
    }
  ]
}
```

At a glance it may look like quite a bit however, it can be thought of as having three sections:

1. File level metadata
1. Command level metadata
1. Variable metadata (things can act on both metadata _and_ data; the use of 'variable metadata' is confusing). "Variable level JSON, etc"

### As ProvONE
We need a fairly abstract representation of the script execution before discussing where SDTL fits in. For more information on this, see this [ProvONE Example](https://github.com/ThomasThelen/provone-examples/tree/master/single-read-write).


What is the difference between a `provone: Execution` and a `provone: Program`? 

A `provone:Execution` is an instantiation of a `provone:Program`. A `provone:Program` is abstract to the level where it doesn't care about the specifics (names) of input and output files, but instead wants to know that there were inputs and outputs. 
It's not until a `provone:Execution` that real inputs and outputs are documented.




In the pictoral representation of the integration below, the objects colored in red are the main focus.

![Alt text](https://i.ibb.co/Bcx0Mgg/generation-added.png)

Note that the command is related to the original file by `hasSubProgram` rather than being connected to the `prov:Execution` with hasPart. This boils down to the philosophical debate of whether the SDTL represents the running of the command, or whether it describes the command itself!

I decided that the SDTL represents the actual command, and is not bound to an actual execution of it.

Mental Excercise: Imagine a script that has a line which assigns a random number to a variable. C2Metadata isn't concerned with _what_ is assigned to the variable; it's concerned that a random variable is assigned to the variable.


### Describing Command/subProgram Metadata
Since each command represents a computational execution they can have type `provone:Program`. This allows us to attach input and output ports with `provone:hasInPort` and `provone:hasOutPort`.

The metadata about the command can be lumped into this object, enhancing its description.


### Describing Variable Metadata


When the command generates _something_, we can represent that _something_ as a `provone:Port`. In this example, the output of the command is a variable `newVar`. The output ports can theoretically be used as input ports to other commands(subPrograms), which allows us to capture the flow of information over the entire execution.

Metadata about outputs _should_ be able to be included in the Port.


Before showing the complete meshing of the SDTL and ProvONE, I thought that it would be helpful to see a visualization of how I combined the two so I included the following 3000x3000 image that shows which object the SDTL is mapped to. 
![Alt text](https://i.imgur.com/J1nvb4l.png)


The complete JSON-LD of ProvONE with SDTL:

```json=
{
   "@context":[
      {
          "prov":"http://www.w3.org/ns/prov#"
      },
      {
          "provone":"http://purl.dataone.org/provone/2015/01/15/ontology#"
      },
      {
          "sdtl": "SDTL URL"
      }
   ],
   "@graph":[
      {
          "@id":"program-1_execution",
          "@type":"provone:Execution",
          "prov:qualifiedAssociation": "associated_plan"
      },
      
      {
          "@id": "associated_plan",
          "@type": "Association",
          "provone:hadPlan": "program-1.spss"
      },
      
      {
          "@id": "program-1.spss",
          "@type": "provone:Program",
          "provone:hasSubProgram": "command_spss",
          "stdl:id": "program-1",
          "stdl:sourceFileName": "",
          "stdl:sourceLanguage": "spss",
          "stdl:scriptMD5": "518001a968c359366bf7ceb12bf209ea",
          "stdl:scriptSHA1": "3dead21a7b31e1409d2ab364cbf4f734366186ad",
          "stdl:sourceFileLastUpdate": "2020-04-14T18:38:10+00:00",
          "stdl:sourceFileSize": 19,
          "stdl:lineCount": 1,
          "stdl:commandCount": 1,
          "stdl:parser": "spss-to-sdtl",
          "stdl:parserVersion": "0.9 Development",
          "stdl:modelVersion": "0.9 Development",
          "stdl:modelCreatedTime": "2020-04-14T18:38:10+00:00"
      },
      
      {
          "@id": "command_spss",
          "@type": "provone:Program",
          "provone:hasOutPort": "generated_var",
          "sdtl:$type": "Compute",
          "sdtl:command": "compute",
          "sdtl:sourceInformation": {
              "sdtl:lineNumberStart": 1,
              "sdtl:lineNumberEnd": 1,
              "sdtl:sourceStartIndex": 1,
              "sdtl:sourceStopIndex": 19,
              "sdtl:originalSourceText": "compute newVar = 0."
          }
      },
      {
          "@id": "generated_var",
          "@type": "provone:Port",
          "stdl:variable": {
            "stdl:$type": "VariableSymbolExpression",
            "sdtl:variableName": "newVar"
          },
          "sdtl:expression": {
            "sdtl:$type":"NumericConstantExpression",
            "sdtl:value": "0",
            "sdtl:numericType": "int"
          }
      }
      
   ]
}
```


### Describing File-Level Metadata
The first chunk of metadata in the SDTL is concerned with file level metrics: checksums, line count, script type, file size, etc.

This type of information is most appropriate in the top level `provone:Program` since it's a description about the general script file.

#### Should be included
These are properties that I feel confident placing in the `provone:Program` that contains the subPrograms.
```json=
          "stdl:id": "program-1",
          "stdl:sourceFileName": "",
          "stdl:sourceLanguage": "spss",
          "stdl:scriptMD5": "518001a968c359366bf7ceb12bf209ea",
          "stdl:scriptSHA1": "3dead21a7b31e1409d2ab364cbf4f734366186ad",
          "stdl:sourceFileLastUpdate": "2020-04-14T18:38:10+00:00",
          "stdl:sourceFileSize": 19,
          "stdl:lineCount": 1,
          "stdl:commandCount": 1,
```

#### Questionable Additions
These properties _could_ be included in the provone:Program with the point of view:
_The provone:Program has attributes from the parser with properties_
```json=
          "stdl:parser": "spss-to-sdtl",
          "stdl:parserVersion": "0.9 Development",
          "stdl:modelVersion": "0.9 Development",
          "stdl:modelCreatedTime": "2020-04-14T18:38:10+00:00",
```

#### Files with Code
If the file has code (which it should) then it's going to have a `provone:hasSubProgram` relation _for every command_ where `command` is what C2Metadata calls a command. The composition of all the `subProgram` relations make up the overall program.

For example, a Python script with two commands can be represented as
![Alt text](https://i.imgur.com/PdpahhP.png)

The main example used in this document has a single command, which is why there's only one `provone:hasSubProgram` relation.