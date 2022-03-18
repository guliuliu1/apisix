---
title: mocking
---

<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

## Description

Mock API plugin, When the plugin is bound, it returns random mock data in the specified format and is no longer forwarded to the upstreams.

## Attributes

| Name            | Type    | Requirement | Default | Valid                                                            | Description                                                                                                                                              |
| -------------   | -------| ----- | ----- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| delay           | integer | optional |        |                                                                 | Delay return time, in seconds                                            |
| response_status | integer| optional  | 200 |                                                                 | response http status code                                            |
| content_type    | string | optional  | application/json |                                                                 | response header Content-Type。                                            |
| response_example| string | optional  |        |                                                                 | response body                                            |
| response_schema | object | optional  |        |                                                                 | The jsonschema object for the response is specified. This property takes effect if the `response_example` is not specified                                            |
| with_mock_header | boolean | optional | true  |                                                                 | Whether to return the response header: "x-mock-by: APISIX/{version}", returned by default, false does not return        |

Supported field types: `string`, `number`, `integer`, `boolean`, `object`, `array`
Base data types (`string`, `number`, `integer`, `Boolean`) through configuration example attribute to specify the generated response value, random return not configured.
Here is a `jsonschema` example:

```json
{
    "properties":{
        "field0":{
            "example":"abcd",
            "type":"string"
        },
        "field1":{
            "example":123.12,
            "type":"number"
        },
        "field3":{
            "properties":{
                "field3_1":{
                    "type":"string"
                },
                "field3_2":{
                    "properties":{
                        "field3_2_1":{
                            "example":true,
                            "type":"boolean"
                        },
                        "field3_2_2":{
                            "items":{
                                "example":155.55,
                                "type":"integer"
                            },
                            "type":"array"
                        }
                    },
                    "type":"object"
                }
            },
            "type":"object"
        },
        "field2":{
            "items":{
                "type":"string"
            },
            "type":"array"
        }
    },
    "type":"object"
}
```

Here are the return objects that might be generated by this `jsonschema`:

```json
{
    "field1": 123.12,
    "field3": {
        "field3_1": "LCFE0",
        "field3_2": {
            "field3_2_1": true,
            "field3_2_2": [
                155,
                155
            ]
        }
    },
    "field0": "abcd",
    "field2": [
        "sC"
    ]
}
```

## How To Enable

Here, use `route` as an example (`service` is used in the same way) to enable the `mocking` plugin on the specified `route`.

```shell
curl http://127.0.0.1:9080/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "methods": ["GET"],
    "uri": "/index.html",
    "plugins": {
        "mocking": {
            "delay": 1,
            "content_type": "application/json",
            "response_status": 200,
            "response_schema": {
               "properties":{
                   "field0":{
                       "example":"abcd",
                       "type":"string"
                   },
                   "field1":{
                       "example":123.12,
                       "type":"number"
                   },
                   "field3":{
                       "properties":{
                           "field3_1":{
                               "type":"string"
                           },
                           "field3_2":{
                               "properties":{
                                   "field3_2_1":{
                                       "example":true,
                                       "type":"boolean"
                                   },
                                   "field3_2_2":{
                                       "items":{
                                           "example":155.55,
                                           "type":"integer"
                                       },
                                       "type":"array"
                                   }
                               },
                               "type":"object"
                           }
                       },
                       "type":"object"
                   },
                   "field2":{
                       "items":{
                           "type":"string"
                       },
                       "type":"array"
                   }
               },
               "type":"object"
           }
        }
    },
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    }
}'
```

## Test Plugin

the `mocking` plugin is configured as follows:

```json
{
  "delay":0,
  "content_type":"",
  "with_mock_header":true,
  "response_status":201,
  "response_example":"{\"a\":1,\"b\":2}"
}
```

Use curl to access:

```shell
$ curl http://127.0.0.1:9080/test-mock -i
HTTP/1.1 201 Created
...
Content-Type: application/json;charset=utf8
x-mock-by: APISIX/2.10.0
...

{"a":1,"b":2}
```

## Disable Plugin

When you want to disable this plugin, it is very simple,
you can delete the corresponding JSON configuration in the plugin configuration,
no need to restart the service, it will take effect immediately:

```shell
curl http://127.0.0.1:9080/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "methods": ["GET"],
    "uri": "/index.html",
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    }
}'
```

This plugin has been disabled now. It works for other plugins.