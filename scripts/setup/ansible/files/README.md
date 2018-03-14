
## Navigation
[Content][] | [Policy Manager Architecture][] | [Python extractor][] | [Policy definition][] | [jsonschema2pojo generator][] | [Seed][] | [Generated Jar][] | [whatchdog daemon][] | [Working][] | [Authors][] | [License][]

This folder will contain some neccessary files to deploy the system:

# Content

* RackSpace_inventory.sh : A script to check which virtual machines are working and their ip addresses
* policy.tar.gz: Policy Manager contents and scripts
* reflector.zip: A Docker image components that read metrics from Monasca machine kafka server and pushes them into general Kafka server
* rsa: Rsa private key generated to read and deploy content automatically from github repositories
* rsa.pub: Rsa public key generated to read and deploy content automatically from github repositories. 

# Policy Manager Architecture

Policy manager is composed of:

* Python extractor
* seed
* jsonschema2pojo generator
* whatchdog daemon
* generated jar

## Policy definition

A new policy will consist on a ECA-SUPA JSON like element. Refer to [Policy Manager](https://github.com/CogNet-5GPPP/Apis_public/tree/master/helloworlds/policy#policy-definition) to get more info.

```
{"supa-policy": {
    "supa-policy-validity-period": {
      "start": "2016-12-20T08:42:57.527404Z"
    },
    "supa-policy-target": {
      "domainName": "systemZ",
      "subNetwork": "192.168.1.1",
      "instanceName": ["devstack", "devstack2"],
      "topicName": "demo1"
    },
    "supa-policy-statement": {
      "event": {
        "event-name": "cpu_performance",
        "event-value-type": "int",
        "event-value": "10",
        "instanceName": ["compute1", "compute2"]

      },
      "condition": {
        "condition-name": "cpu_performance_high",
        "condition-operator": ">",
        "condition-threshold": "95"
      },
      "action": {
        "action-name": "deploy_new_machines",
        "action-host": "http://host.com/",
        "action-type": "deploy-topology",
        "action-param":[ {
          "param-type": "topology",
          "param-value": "ring",
          "instanceName": ["compute1", "compute2"]
        },{
          "param-type": "size",
          "param-value": "10",
          "instanceName": ["compute1", "compute2"]
        }]
      }
    }
  }
}
```


## Python extractor

Python extractor is the main element of this architecture, it consists on a Python script that reads *newpolicy* topic from *Kafka* server and waits for a new *ECA-SUPA* policy definition message. After receiving it, executes some tasks to build a new jar that will be the responsible of listening the *topicName* topic from *Kafka* that was set on the policy definition and executes an action if some conditions are fulfiled.

This module extracts some elements from *ECA-SUPA* newpolicy:

* topicName
* condition-name
* condition-operator
* condition-threshold
* action-param
* action-host

and fills the *seed* module with those values.


## jsonschema2pojo generator


[jsonschema2pojo](http://www.jsonschema2pojo.org/) is a Java tool that reads *JSON* files  and creates Java [POJO](https://spring.io/understanding/POJO) classes.

## Seed

It is a predefined *Kafka consumer* Java based implementation.
It contains several elements that must be filled (package name,etc) by python extractor.
It builds with maven.
It generates an executable *jar* file.

After filling this module with the elements extracted from Python extractor and added with the class generated by jsonschema2pojo the system builds it with maven. It generates a jar file that will be the responsible to look for new *ECA-SUPA* messages to be checked.

## Generated Jar

This autogenerated jar file will be the responsible to listen for new measures from Kafka and checking if those measures fulfil the required condition.

This module will listen for the topic *topicName* in the *Kafka* server.

The system will generate a jar for each combination of ```topicName``` and ```condition-name```. This means that the *Policy Manager* can await for diferent condition names in the same *topicName* topic from *Kafka* server.

## whatchdog daemon

It is a bash based script that whatches a directory for the new *jar* autogenerated files and executes them.

## Working

1. Python extractor reads topic *newpolicy* from kafka.
2. When a new policy message comes, it extracts some elements from it and saves *ECA-SUPA* on a json file.
3. Next step will be the generation a *POJO* class from that json policy using jsonschema2pojo.
4. That class will be copied to complete the *seed* and the system will fill the *seed* with the appropriate params (* topicName,condition-name,condition-operator,condition-threshold,action-param,action-host).
5. The system will build the *seed* Java project using maven and copy the generated jar file to the directory that is been whatched by whatchdog daemon.
6. Whatchdog daemon will launch new jar file.
7. New generated jar file will listen *topicName+condition-name* *Kafka* topic for new messages to be evaluated.


# Reflector

This Docker image will be the responsible of "reading" measures from the *Kafka* server from the *Monasca* virtual machine and "reflect" them to the *Kafka* machine, making them available to all elements of the *CogNet* infrastructure.

### Authors


- Angel Martin (amartin@vicomtech.org)
- Felipe Mogollon (fmogollon@vicomtech.org)

### License


Copyright 2016 Vicomtech.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0.

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

[Content]: #content
[Policy definition]: #policy-definition
[Policy Manager Architecture]: #policy-manager-architecture
[Python extractor]: #python-extractor
[jsonschema2pojo generator]: #jsonschema2pojo-generator
[Seed]: #seed
[Generated Jar]: #generated-jar
[whatchdog daemon]: #whatchdog-daemon
[Reflector]: #reflector
[Working]: #working
[Authors]: #authors
[License]: #license