# BIG-IQ and CIS Integration Quick Start Guide

**Note** This integration of **BIG-IQ and CIS** is for early field trials/experimental and demo purpose. We would like to get some feedback and potentially make improvements. The integration has not gone through regression testing. This page provides a how-to for BIG-IQ integration with CIS and BIG-IP implementation. Please open issues on my github page or contact me at m.dittmer@f5.com.

# Note
This document assumes the user already have a knowledge of setting up CIS Controller.
Please visit to F5 Cloud Docs for more information.

Environment parameters:
* K8s Version: 1.13
* BIG-IQ Version: 7.0.0
* BIG-IP Version: 13.1.1
* CIS Image:: snatra27/cis-k8s-ctlr-biq-as3:1.0

## Prerequisite
Since we are using BIG-IQ, BIG-IQ will install AS3 on BIG-IP

## Create kubernetes bigip container ingress service

Located the deployment file in the deployment-artifacts folder

## Create ConfigMap
This config map contains both BIG-IQ AS3 template and BIG-IQ AS3 deployment.
Users can modify the template according to their requirement and also users can remove the "BIQ_AS3_TEMPLATE" if already have pre-configured template available on BIG-IQ system. However, user must configure "BIQ_AS3_DECELERATION".
and CIS will only supports single entry of "BIQ_AS3_DECELERATION" at this point of time.

```
apiVersion: v1
data:
  template: |
{
  "BIQ_AS3_TEMPLATE": {
    "name": "CIS-AS3-BIQ-BIGIP-TEMPLATE",
    "published": true,
    "description": "BIQ AS3 Template for CIS and BIG IQ Integration",
    "schemaOverlay": {
      "type": "object",
      "properties": {
        "class": {
          "type": "string",
          "const": "Application"
        },
        "template": {},
        "schemaOverlay": {},
        "label": {},
        "remark": {}
      },
      "additionalProperties": {
        "allOf": [
          {
            "anyOf": [
              {
                "properties": {
                  "class": {
                    "const": "Analytics_Profile"
                  }
                }
              },
              {
                "properties": {
                  "class": {
                    "const": "Pool"
                  }
                }
              },
              {
                "properties": {
                  "class": {
                    "const": "Service_HTTP"
                  }
                }
              }
            ]
          },
          {
            "if": {
              "properties": {
                "class": {
                  "const": "Analytics_Profile"
                }
              }
            },
            "then": {
              "$ref": "#/definitions/Analytics_Profile"
            }
          },
          {
            "if": {
              "properties": {
                "class": {
                  "const": "Pool"
                }
              }
            },
            "then": {
              "$ref": "#/definitions/Pool"
            }
          },
          {
            "if": {
              "properties": {
                "class": {
                  "const": "Service_HTTP"
                }
              }
            },
            "then": {
              "$ref": "#/definitions/Service_HTTP"
            }
          }
        ]
      },
      "required": [
        "class"
      ],
      "definitions": {
        "Analytics_Profile": {
          "properties": {
            "class": {},
            "collectUserAgent": {
              "type": "boolean"
            },
            "collectClientSideStatistics": {
              "type": "boolean",
              "default": true
            },
            "collectGeo": {
              "type": "boolean"
            },
            "collectUrl": {
              "type": "boolean"
            },
            "collectPageLoadTime": {
              "type": "boolean"
            },
            "collectOsAndBrowser": {
              "type": "boolean",
              "default": false
            },
            "collectMethod": {
              "type": "boolean",
              "default": false
            },
            "collectResponseCode": {
              "type": "boolean",
              "default": true
            },
            "collectIp": {
              "type": "boolean"
            }
          },
          "type": "object",
          "additionalproperties": false
        },
        "Pool": {
          "properties": {
            "class": {},
            "members": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "servicePort": {
                    "type": "number",
                    "default": 80
                  },
                  "monitors": {
                    "type": "array",
                    "default": [
                      "http"
                    ],
                    "const": [
                      "http"
                    ]
                  },
                  "adminState": {
                    "type": "string",
                    "default": "enable"
                  },
                  "shareNodes": {
                    "type": "boolean",
                    "default": true,
                    "const": true
                  },
                  "serverAddresses": {
                    "type": "array"
                  }
                }
              }
            },
            "monitors": {
              "type": "array",
              "default": [
                "http"
              ],
              "const": [
                "http"
              ]
            }
          },
          "type": "object",
          "additionalproperties": false
        },
        "Service_HTTP": {
          "properties": {
            "class": {},
            "virtualPort": {
              "type": "number",
              "default": 80
            },
            "profileAnalytics": {
              "type": "object",
              "properties": {
                "use": {
                  "type": "string",
                  "default": "Analytics_Profile"
                }
              }
            },
            "virtualAddresses": {
              "type": "array"
            },
            "pool": {
              "type": "string",
              "default": "Pool"
            },
            "enable": {
              "type": "boolean",
              "default": true
            }
          },
          "type": "object",
          "additionalproperties": false
        }
      }
    }
  },
  "BIQ_AS3_DECLERATION": {
    "applicationName": "CIS-BIQ-AS3-DEMO",
    "applicationDescription": "CIS Integration with BIG IQ Demo Application",
    "appSvcsDeclaration": {
      "class": "AS3",
      "persist": true,
      "declaration": {
        "class": "ADC",
        "schemaVersion": "3.12.0",
        "target": {
          "address": "10.145.72.109"
        },
        "cis": {
          "class": "Tenant",
          "BIGIQAS3": {
            "class": "Application",
            "schemaOverlay": "CIS-AS3-BIQ-BIGIP-TEMPLATE",
            "template": "http",
            "serviceMain": {
              "virtualPort": 80,
              "profileAnalytics": {
                "use": "Analytics_Profile"
              },
              "virtualAddresses": [
                "172.16.3.29"
              ],
              "pool": "Pool",
              "enable": true,
              "class": "Service_HTTP"
            },
            "Analytics_Profile": {
              "collectUserAgent": false,
              "collectClientSideStatistics": true,
              "collectGeo": false,
              "collectUrl": false,
              "collectPageLoadTime": false,
              "collectOsAndBrowser": false,
              "collectMethod": false,
              "collectResponseCode": true,
              "collectIp": false,
              "class": "Analytics_Profile"
            },
            "Pool": {
              "members": [
                {
                  "servicePort": 8080,
                  "adminState": "enable",
                  "serverAddresses": []
                }
              ],
              "class": "Pool"
            }
          }
        }
      }
    }
  }
}
kind: ConfigMap
metadata:
  creationTimestamp: "2019-09-16T15:32:46Z"
  labels:
    as3: "true"
    f5type: virtual-server
  name: as3-declaration
  namespace: kube-system
```

On successful deployment of ConfigMap, user must able to view an healthy application created with given name( in this example: "CIS-BIQ-AS3-DEMO") on BIG-IQ GUI.
  

