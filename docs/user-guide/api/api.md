---
layout: default
title: TerminusDB API
parent: User guide
nav_order: 5
has_children: true
permalink: /docs/user-guide/api
---

# TerminusDB API
{: .no_toc }

The TerminusDB Server includes a built in HTTP server which implements the Terminus API consisting of 12 endpoints:

- connect - GET http://terminus.db/
- create_database - POST http://terminus.db/<account>/<dbid>
- delete_database - DELETE http://terminus.db/<account>/<dbid>
- get_schema - GET http://terminus.db/schema/<account>/<dbid>/<repo>/branch/<branchid>/[<schema_graphid>]
- update_schema - POST http://terminus.db/schema/<account>/<dbid>/local/branch/<branchid>/[<schema_graphid>]
- class_frame - GET http://terminus.db/frame/<account>/<dbid>/<repo>/branch/<branchid>
- clone - POST http://terminus.db/clone/<account>/[<new_dbid>]
- fetch - POST http://terminus.db/fetch/<account>/<dbid>/<repo_id>
- rebase - POST http://terminus.db/rebase/<account>/<dbid>/<repo>/<branchid>/[<remote_repo_id>]/[<remote_branch_id>]
- push - POST http://terminus.db/push/<account>/<dbid>/<repo>/<branchid>/[<remote_repo_id>]/[<remote_branch_id>]
- branch - POST http://terminus.db/branch/<account>/<dbid>/<repo>/<new_branchid>
- create graph - POST http://terminus.db/graph/<account>/<dbid>/<repo>/branch/<branchid>/<instance|schema|inference>/<graphid>
- delete graph - DELETE http://terminus.db/graph/<account>/<dbid>/<repo>/branch/<branchid>/<instance|schema|inference>/<graphid>
- woql_select - GET http://terminus.db/<account>/<dbid>/woql
- woql_update - POST http://terminus.db/<account>/<dbid>/woql
- metadata - GET http://terminus.db/<account>/<dbid>/metadata

The terminus administration schema ( http://terminusdb.com/schema/terminus ) contains definitions for all of the data structures and properties used in the API. All arguments and returned messages are encoded as JSON.

![](img/screenshots/terminus_client/api_calls.jpg)

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Connect

Connects to a TerminusDB Server and receives a Capability Document which defines the client's permissions on the server.

### Example

```
GET http://terminus.db/ curl --user 'username:password' "http://localhost"
```

### Arguments

The username and password for the client to connect to the server needs to be supplied as basic auth.
Depending if the server has been compiled with JWT support, the authentication can be a JWT token
with the `https://terminusdb.com/nickname` key provided as the username.

### Return

A document of type terminus:Capability (or one of its subclasses) http://terminusdb.com/schema/terminus#Capability

```json
"@context": {
    "doc":"http://localhost/terminus/document/",
    "owl":"http://www.w3.org/2002/07/owl#",
    "rdf":"http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs":"http://www.w3.org/2000/01/rdf-schema#",
    "terminus": "http://terminusdb.com/schema/terminus#"
  },
  "@id":"doc:admin",
  "@type":"terminus:User",
  "rdfs:comment": {"@language":"en", "@value":"This is the server super user account"},
  "rdfs:label": {"@language":"en", "@value":"Server Admin User"},
  "terminus:authority": {
    "@id":"doc:access_all_areas",
    "@type":"terminus:ServerCapability",
    "rdfs:comment": {"@language":"en", "@value":"Access all server functions"},
    "rdfs:label": {"@language":"en", "@value":"All Capabilities"},
    "terminus:action": [
      {"@id":"terminus:class_frame", "@type":"terminus:DBAction"},
      {"@id":"terminus:create_database", "@type":"terminus:DBAction"},
      {"@id":"terminus:create_document", "@type":"terminus:DBAction"},
      {"@id":"terminus:delete_database", "@type":"terminus:DBAction"},
      {"@id":"terminus:delete_document", "@type":"terminus:DBAction"},
      {"@id":"terminus:get_document", "@type":"terminus:DBAction"},
      {"@id":"terminus:get_schema", "@type":"terminus:DBAction"},
      {"@id":"terminus:update_document", "@type":"terminus:DBAction"},
      {"@id":"terminus:update_schema", "@type":"terminus:DBAction"},
      {"@id":"terminus:woql_select", "@type":"terminus:DBAction"},
      {"@id":"terminus:woql_update", "@type":"terminus:DBAction"}
    ],
    "terminus:authority_scope": [
      {
        "@id":"doc:dima",
        "@type":"terminus:Database",
        "rdfs:comment": {
          "@language":"en",
          "@value":"This is a DB created for Dima to test the ontoogies"
        },
        "rdfs:label": {"@language":"en", "@value":"Dima Test DB"},
        "terminus:allow_origin": {"@type":"xsd:string", "@value":"*"},
        "terminus:id": {
          "@type":"xsd:anyURI",
          "@value":"http://localhost/dima"
        },
        "terminus:instance": {"@type":"xsd:string", "@value":"document"},
        "terminus:schema": {"@type":"xsd:string", "@value":"schema"}
      },
      {
        "@id":"doc:documentation",
        "@type":"terminus:Database",
        "rdfs:comment": {"@language":"en", "@value":"This is the documentation db"},
        "rdfs:label": {"@language":"en", "@value":"Documentation database"},
        "terminus:allow_origin": {"@type":"xsd:string", "@value":"*"},
        "terminus:id": {
          "@type":"xsd:anyURI",
          "@value":"http://localhost/documentation"
        },
        "terminus:instance": {"@type":"xsd:string", "@value":"document"},
        "terminus:schema": {"@type":"xsd:string", "@value":"schema"}
      },
      {
        "@id":"doc:terminus",
        "@type":"terminus:Database",
        "rdfs:comment": {
          "@language":"en",
          "@value":"The master database contains the meta-data about databases, users and roles"
        },
        "rdfs:label": {"@language":"en", "@value":"Master Database"},
        "terminus:allow_origin": {"@type":"xsd:string", "@value":"*"},
        "terminus:id": {
          "@type":"xsd:anyURI",
          "@value":"http://localhost/terminus"
        }
      },
      {
        "@id":"doc:server",
        "@type":"terminus:Server",
        "rdfs:comment": {
          "@language":"en",
          "@value":"The current Database Server itself"
        },
        "rdfs:label": {"@language":"en", "@value":"The DB server"},
        "terminus:allow_origin": {"@type":"xsd:string", "@value":"*"},
        "terminus:id": {"@type":"xsd:anyURI", "@value":"http://localhost"},
        "terminus:resource_includes": [
          {"@id":"doc:dima", "@type":"terminus:Database"},
          {"@id":"doc:documentation", "@type":"terminus:Database"},
          {"@id":"doc:terminus", "@type":"terminus:Database"}
        ]
      }
    ]
  }
}
```


## Create Database

    POST [http://terminus.db/](http://terminus.db/DBNAME)db/<account>/<dbid>


### Arguments

Post argument is a JSON document of the following form

    { <"base_uri" : MY_BASE_DOCUMENT_URI>,
      "label" : "A Label", 
      "comment" : "A Comment" 
    }

Base_URI will default to "http://hub.terminusdb.com/account/db/document/"

terminus://dbid/

Hub create DB will POST [http://terminus.db/](http://terminus.db/DBNAME)db/account/dbid with

    { "base_uri": "http://hub.terminusdb.com/account/db"
    }

### Return

A terminus result message indicating either terminus:success or terminus:failure

`{"terminus:status":"terminus:success"}`


## Delete Database

Deletes an entire Database

Sends a HTTP DELETE request to the URL of the Database on a Terminus Server

```
DELETE http://terminus.db/db/<account>/<dbid>

curl --user ":secret_key" -X "DELETE" http://localhost/tcre`
```

### Return

A terminus result message indicating either terminus:success or terminus:failure

`{"terminus:status":"terminus:success"}`


## Create Document

Create a document by sending a JSON-LD document which conforms to the database schema, to the Terminus API.

```
POST http://localhost/DBID/document/DOCID`

curl --user ":secret_key" -d "@my-doc.json" -X POST http://localhost/dima/document/terminusAPI
```

### Argument

A JSON-LD document of a valid document type for the given database, wrapped in a terminus:APIUpdate document.

```
{
   "@context":
   {
         "doc":"http://195.201.12.87:6363/dima/document/",
         "owl":"http://www.w3.org/2002/07/owl#",
         "rdf":"http://www.w3.org/1999/02/22-rdf-syntax-ns#",
         "rdfs":"http://www.w3.org/2000/01/rdf-schema#",
         "xsd":"http://www.w3.org/2001/XMLSchema#"
    },
    "terminus:document":{
        "rdfs:label":[{"@value":"The Terminus API itself","@type":"xsd:string"}],
        "rdfs:comment":[{"@value":"here we go","@type":"xsd:string"}],
        "@type":"http://terminusdb.com/schema/documentation#APIDefinition",
        "@id":"http://localhost/dima/document/terminusAPI"
     },
     "@type":"terminus:APIUpdate",
}
```

### Return

A terminus result message indicating either terminus:success or terminus:failure

`{"terminus:status":"terminus:success"}`


## Get Document

Retrieves a terminus document in JSON-LD form, from a terminus server

`GET --user "username:secret_key" http://localhost/terminus/document/server`

### Arguments

The terminus:encoding parameter can be either terminus:jsonld or terminus:frame - both are json-ld representations.

    terminus:encodingterminus:jsonld | terminus:frame

### Return

A JSON-LD document representing the requested item

     "@context": {
        "doc":"http://localhost/terminus/document/",
        "owl":"http://www.w3.org/2002/07/owl#",
        "rdf":"http://www.w3.org/1999/02/22-rdf-syntax-ns#",
        "rdfs":"http://www.w3.org/2000/01/rdf-schema#",
        "terminus":"http://terminusdb.com/schema/terminus#",
        "xsd":"http://www.w3.org/2001/XMLSchema#"
      },
      "@id":"doc:server",
      "@type":"terminus:Server",
      "rdfs:comment": {"@language":"en", "@value":"The current Database Server itself"},
      "rdfs:label": {"@language":"en", "@value":"The DB server"},
      "terminus:allow_origin": {"@type":"xsd:string", "@value":"*"},
      "terminus:resource_includes": [
        {"@id":"doc:dbWhichIAmGoingToDelete", "@type":"terminus:Database"},
        {"@id":"doc:dima", "@type":"terminus:Database"},
        {"@id":"doc:documentation", "@type":"terminus:Database"},
        {"@id":"doc:documentaton", "@type":"terminus:Database"},
        {"@id":"doc:dummy", "@type":"terminus:Database"},
        {"@id":"doc:terminus", "@type":"terminus:Database"},
        {"@id":"doc:test", "@type":"terminus:Database"}
      ]
    }


## Update Document

Updates a document by sending a new version to the api endpoint - all document updates are atomic - the entire document is replaced with the passed version.

`POST http://localhost/DBID/document/DOCID`

  `curl --user 'username::secret_key'  -d "@my-doc.json" -X POST http://localhost/dima/document/terminusAPI`


### Argument

The document in JSON-LD format

    {
          "@context": {
                    "doc":"http://localhost/dima/document/",
                    "owl":"http://www.w3.org/2002/07/owl#",
                    "rdf":"http://www.w3.org/1999/02/22-rdf-syntax-ns#",
                    "rdfs":"http://www.w3.org/2000/01/rdf-schema#",
                    "xdd":"https://datachemist.net/ontology/xdd#",
                    "xsd":"http://www.w3.org/2001/XMLSchema#"
           },
           "terminus:document":{
              "rdfs:label":[{"@value":"The Terminus API itself","@type":"xsd:string"}],
              "rdfs:comment":[{"@value":"here we goaaaa","@type":"xsd:string"}],
              "@type":"http://terminusdb.com/schema/documentation#APIDefinition",
              "@id":"http://localhost/dima/document/terminusAPI"
           },
          "@type":"terminus:APIUpdate"
    }

### Return

A terminus result message indicating success or failure

      {"terminus:status":"terminus:success"}

## Delete Document

Send a HTTP delete to the Document URL

    curl --user ':secret_key'  -X "DELETE" http://localhost/dima/document/terminusAPI

### Return

A terminus result message indicating either terminus:success or terminus:failure

      {"terminus:status":"terminus:success"}


## Get Schema

Retrieves the database schema as a turtle encoding from the database `GET http://localhost/DBID/schema`

    curl --user ':secret_key'  http://localhost/dima/schema?terminus:encoding=terminus:turtle

### Arguments:

    terminus:encodingtterminus:turtle

### Return

The database schema encoded as a JSON string containing the contents of a turtle file

    "@prefix rdf: pre>http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
    @prefix
        ... OWL schema serialised as turtle ...
    "


## Update Schema

Updates the schema by posting a new schema version to the database API. Schema updates are atomic - the entire schema is replaced by the updated version

`POST http://localhost/dima/schema`

    curl --user ':secret_key'  -d "@my-schema.ttl" -X POST http://localhost/dima/document/terminusAPI

### Argument

A terminus:APIUpdate document with the contents of the turtle held by the terminus:turtle predicate.

    {
                "terminus:turtle": "@prefix rdf:
                                                ... OWL schema serialised as turtle ...",
                "terminus:schema":"schema",
                "@type":"terminus:APIUpdate"
    }

### Return

A terminus result message indicating either terminus:success or terminus:failure

      {"terminus:status":"terminus:success"}


## Class Frame

Retrieves a frame representation of a class within the ontology - a json representation of all the logic contained in the class.

`GET http://terminus.db/DBID/frame`

    curl --user ':secret_key' "http://localhost/dima/schema?terminus:class=
    http://terminusdb.com/schema/documentation#APIEndpointSpecification"

### Argument

The class that the frame is for must be passed in the terminus:class property

    terminus:classthttp://terminusdb.com/schema/documentation#APIEndpointSpecification
    terminus:user_keytsecret

### Return

An array of frames, each of which is encoded as a JSON-LD frame document and each of which represents a single property in the class frame.

    [
      {
        "@context": {
          "doc":"http://localhost/dima/document/",
          "docs":"http://terminusdb.com/schema/documentation#",
          "owl":"http://www.w3.org/2002/07/owl#",
          "rdf":"http://www.w3.org/1999/02/22-rdf-syntax-ns#",
          "rdfs":"http://www.w3.org/2000/01/rdf-schema#",
          "terminus":"http://terminusdb.com/schema/terminus#",
          "xsd":"http://www.w3.org/2001/XMLSchema#"
        },
        "domain":"docs:APIEndpointSpecification",
        "frame": {
          "operands": [
    [
      {
        "domain":"docs:APIReturnValue",
        "property":"docs:type",
        "range":"xsd:string",
        "restriction":"true",
        "type":"datatypeProperty"
      },
      {
        "domain":"docs:APIReturnValue",
        "property":"docs:summary",
        "range":"xsd:string",
        "restriction":"true",
        "type":"datatypeProperty"
      },
      {
        "domain":"docs:APIReturnValue",
        "property":"docs:name",
        "range":"xsd:string",
        "restriction":"true",
        "type":"datatypeProperty"
      },
      {
        "domain":"docs:APIReturnValue",
        "frame": {
          "elements": [
    {
      "class":"docs:json"
    },
    {
      "class":"docs:text"
    },
    {
      "class":"docs:jsonld"
    },
    {
      "class":"docs:css"
    },
    {
      "class":"docs:ttl"
    },
    {
      "class":"docs:html"
    },
    {
      "class":"docs:owl"
    },
    {
      "class":"docs:csv"
    }
          ],
          "type":"oneOf"
        },
        "property":"docs:encoding",
        "range":"docs:DataLanguage",
        "restriction":"true",
        "type":"objectProperty"
      },
      {
        "domain":"docs:APIReturnValue",
        "property":"docs:description",
        "range":"xsd:string",
        "restriction":"true",
        "type":"datatypeProperty"
      }
    ]
      },
      {
        "@context": {
          "docs":"http://terminusdb.com/schema/documentation#",
          "owl":"http://www.w3.org/2002/07/owl#",
          "rdf":"http://www.w3.org/1999/02/22-rdf-syntax-ns#",
          "rdfs":"http://www.w3.org/2000/01/rdf-schema#",
          "xsd":"http://www.w3.org/2001/XMLSchema#"
        },
        "domain":"docs:APIEndpointSpecification",
        "property":"rdfs:comment",
        "range":"xsd:string",
        "restriction":"true",
        "type":"datatypeProperty"
      }
    ]


## WOQL Select

To be documented

## WOQL Update

To be documented

## Metadata

`GET http://terminus.db/DBNAME/metadata`

The metadata associated with a database can be retrieved with a GET to the `metadata`.

    curl --user 'username:secret_key' -X GET -H 'Content-Type: application/json' 'http://localhost:6363/terminus/metadata' 

### Return

A `terminus:DatabaseMetadata` object is returned whose structure is as follows:

    {
      "@context": {
        "doc":"http://localhost:6363/terminus/document/",
        "owl":"http://www.w3.org/2002/07/owl#",
        "rdf":"http://www.w3.org/1999/02/22-rdf-syntax-ns#",
        "rdfs":"http://www.w3.org/2000/01/rdf-schema#",
        "scm":"http://localhost:6363/terminus/schema#",
        "tbs":"http://terminusdb.com/schema/tbs#",
        "tcs":"http://terminusdb.com/schema/tcs#",
        "terminus":"http://terminusdb.com/schema/terminus#",
        "vio":"http://terminusdb.com/schema/vio#",
        "xdd":"http://terminusdb.com/schema/xdd#",
        "xsd":"http://www.w3.org/2001/XMLSchema#"
      },
      "@type":"terminus:DatabaseMetadata",
      "terminus:database_created_time": {
        "@type":"xsd:dateTime",
        "@value":"2019-09-30T09:42:26+00:00"
      },
      "terminus:database_modified_time": {"@type":"xsd:dateTime", "@value":"2019-09-30T09:42:26+00:00"},
      "terminus:database_size": {"@type":"xsd:nonNegativeInteger", "@value":113781}
    }


## Clone

POST [http://terminus.db/](http://terminus.db/DBNAME)clone/<account>/[<new_dbid>]

### Arguments

The payload is the **resource identifier of** repo / db that we want to clone. If the new_dbid is provided, this id will be used locally to refer to the DB, otherwise whatever the cloned one uses will be used. 

    {
       @type: "terminus:APIUpdate"
       terminus:resource: URI_OF_RESOURCE_ID
    }

## Fetch

    POST http://terminus.db/fetch/<account>/<dbid>/<repo_id>

POST is empty

    POST http://terminus.db/fetch/<account>/<dbid>/ = http://terminus.db/fetch/<dbid>/origin

## Rebase

    POST http://terminus.db/rebase/<account>/<dbid>/<repo>/<branchid>/[<remote_repo_id>]/[<remote_branch_id>]

Merges deltas from remote_repo_id into dbid/branchid

POST is **empty**

Rebases into dbid/branchid

    POST http://terminus.db/rebase/<account>/<dbid>/<repo>/<branchid> = http://terminus.db/rebase/<dbid>/<branchid>/origin/master

## Push

    POST http://terminus.db/push/<account>/<dbid>/<repo>/<branchid>/[<remote_repo_id>]/[<remote_branch_id>]

Pushes deltas from dbid / branchid the remote repo

POST is empty

e.g.

    http://terminus.db/push/dbid = http://terminus.db/push/dbid/master/origin/master


## Branch

POST http://terminus.db/branch/<account>/<dbid>/<repo>/<new_branchid>

Creates a new branch with parent dbid/new_branchid

### Arguments

POST is a **terminus:Ref resource ID** specifying the base of the new branch to be created.

    {
       "@type" : "terminus:APIUpdate"
       "terminus:resource" : URI_OF_REF_RESOURCE_ID
    }

## Create graph

POST http://terminus.db/graph/<account>/<dbid>/<repo>/branch/<branchid>/<instance|schema|inference>/<graphid>

### Arguments

This takes a post parameter:

    {"commit_info" : { "author" : Author, "message" : Message }}

## Delete graph

    DELETE http://terminus.db/graph/<account>/<dbid>/<repo>/branch/<branchid>/<instance|schema|inference>/<graphid>

### Arguments

This takes a post parameter:

    {"commit_info" : { "author" : Author, "message" : Message }}
