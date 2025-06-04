---
internal: true
---

# Playback Config Examples

The playback service and features available to clients are configured via the [Playback Configuration API](../reference/Playback-Configuration-API.yaml).

Client onboarding is managed through client configuration which is managed in the AWS DynamoDB table 'playback-api-production-clients-configs' and holds
information for a client specific for playback such as:

Here are some working examples of current LIVE Playback Configurations. Note that Playback is moduler so you can add many additonal apotional settings to the mandatory setup.

Examples:
- Cloudpay Playback - Mandatory
- Cloudpay Fusion - Mandatory
- Cloudpay Playback - Tokenized
- Cloudpay Playback - Bitmovin Analytics
- Cloudpay Playback - Adaptive
- Cloudpay Playback - Ads


## Cloudpay Playback - Mandatory

```json
{
    "description": "Cloudpay Mandatory",
    "tenantId": "{Unique-CloudMatrix}",
    "tags": [
        "A Tag"
    ],
    "auth": {
        "cloudpay": {
            "site": "{siteName}-payments.streamamg.com"
        }
    },
    "domains": [
        "https://playback-sdk-demo.dev.streamamg.com",
        "https://playback-sdk-demo.qa.streamamg.com",
        "https://playback-sdk-demo.staging.streamamg.com",
        "https://playback-sdk-demo.streamamg.com"
    ],
    "entitlementsUrl": "https://{CloudMatrixURL}/api/v1/entryentitlements/{entryid}",
    "platform": {
        "kaltura": {
            "partnerId": "123456",
            "partnerAdminSecret": "123456789123456789123"
        }
    },
    "player": {
        "bitmovin": {
            "license": "11111111-2222-3333-4444-555555555555",
            "integrations": {
                "mux": {
                    "player_name": "bitmovin-{name}",
                    "env_key": "1235aavjadwdw11111d1n"
                },
                "resume": {
                    "enabled": true,
                    "jwtSecretKey": "{random UUID}"
                }
            }
        }
    },
    "defaults": {
        "auth": "cloudpay",
        "player": "bitmovin",
        "platform": "kaltura"
    },
    "entitlements": {
        "defaultEntryEntitlement": [
            "{CloudMatrix-Configuration}"
        ]
    },
}
```
## Fusion Playback - Mandatory
```json
{
    "description": "Cloudpay Mandatory",
    "tenantId": "{Unique-CloudMatrix}",
    "tags": [
        "A Tag"
    ],
    "domains": [
        "https://playback-sdk-demo.dev.streamamg.com",
        "https://playback-sdk-demo.qa.streamamg.com",
        "https://playback-sdk-demo.staging.streamamg.com",
        "https://playback-sdk-demo.streamamg.com"
    ],
    "player": {
        "bitmovin": {
            "license": "11111111-2222-3333-4444-555555555555",
            "integrations": {
                "mux": {
                    "player_name": "bitmovin-{name}",
                    "env_key": "1235aavjadwdw11111d1n"
                }
            }
        }
    },
    "defaults": {
        "auth": "jwtsso",
        "player": "bitmovin",
        "entitlements": "fusion"
    },
    "entitlements": {
        "defaultEntryEntitlement": [
            "{CloudMatrix-Configuration}"
        ]
    },
}
```

## Playback Cloudpay - Tokenized
```json
{
    "description": "Cloudpay Tokenizer",
    "tenantId": "{Unique-CloudMatrix}",
    "tags": [
        "A Tag"
    ],
    "auth": {
        "cloudpay": {
            "site": "{siteName}-payments.streamamg.com"
        }
    },
    "domains": [
        "https://playback-sdk-demo.dev.streamamg.com",
        "https://playback-sdk-demo.qa.streamamg.com",
        "https://playback-sdk-demo.staging.streamamg.com",
        "https://playback-sdk-demo.streamamg.com"
    ],
    "deliveryConfiguration": {
        "tokenizer": {
            "streamamg-test.akamaized.net": {
                "key": "{secure token by video team}",
                "tokenName": "hdnts",
                "type": "akamai",
                "windowSeconds": 60
            }
        }
    },
    "entitlementsUrl": "https://{CloudMatrixURL}/api/v1/entryentitlements/{entryid}",
    "platform": {
        "kaltura": {
            "partnerId": "123456",
            "partnerAdminSecret": "123456789123456789123"
        }
    },
    "player": {
        "bitmovin": {
            "license": "11111111-2222-3333-4444-555555555555",
            "integrations": {
                "mux": {
                    "player_name": "bitmovin-{name}",
                    "env_key": "1235aavjadwdw11111d1n"
                },
                "resume": {
                    "enabled": true,
                    "jwtSecretKey": "{random UUID}"
                }
            }
        }
    },
    "defaults": {
        "auth": "cloudpay",
        "player": "bitmovin",
        "platform": "kaltura"
    },
    "entitlements": {
        "defaultEntryEntitlement": [
            "{CloudMatrix-Configuration}"
        ]
    },
}
```

Note: to apply this to fusion, you can use the exact same `deliveryConfiguration` object in the `Mandatory Fusion Config`

## Playback - Bitmovin Analytics
```json
{
    "description": "Cloudpay Bitmovin-Analytics",
    "tenantId": "{Unique-CloudMatrix}",
    "tags": [
        "A Tag"
    ],
    "auth": {
        "cloudpay": {
            "site": "{siteName}-payments.streamamg.com"
        }
    },
    "domains": [
        "https://playback-sdk-demo.dev.streamamg.com",
        "https://playback-sdk-demo.qa.streamamg.com",
        "https://playback-sdk-demo.staging.streamamg.com",
        "https://playback-sdk-demo.streamamg.com"
    ],
    "entitlementsUrl": "https://{CloudMatrixURL}/api/v1/entryentitlements/{entryid}",
    "platform": {
        "kaltura": {
            "partnerId": "123456",
            "partnerAdminSecret": "123456789123456789123"
        }
    },
    "player": {
        "bitmovin": {
            "license": "11111111-2222-3333-4444-555555555555",
            "integrations": {
                "analytics": {
                    "key": "abc12iaaaccsid231a8qfp"
                },
                "resume": {
                    "enabled": true,
                    "jwtSecretKey": "{random UUID}"
                }
            }
        }
    },
    "defaults": {
        "auth": "cloudpay",
        "player": "bitmovin",
        "platform": "kaltura"
    },
    "entitlements": {
        "defaultEntryEntitlement": [
            "{CloudMatrix-Configuration}"
        ]
    },
}
```

## Playback - Adaptive
```json
{
    "description": "Cloudpay Adaptive",
    "tenantId": "{Unique-CloudMatrix}",
    "tags": [
        "A Tag"
    ],
    "auth": {
        "cloudpay": {
            "site": "{siteName}-payments.streamamg.com"
        }
    },
    "domains": [
        "https://playback-sdk-demo.dev.streamamg.com",
        "https://playback-sdk-demo.qa.streamamg.com",
        "https://playback-sdk-demo.staging.streamamg.com",
        "https://playback-sdk-demo.streamamg.com"
    ],
    "entitlementsUrl": "https://{CloudMatrixURL}/api/v1/entryentitlements/{entryid}",
    "platform": {
        "kaltura": {
            "partnerId": "123456",
            "partnerAdminSecret": "123456789123456789123"
        }
    },
    "player": {
        "bitmovin": {
            "license": "11111111-2222-3333-4444-555555555555",
            "integrations": {
                "mux": {
                    "player_name": "bitmovin-{name}",
                    "env_key": "1235aavjadwdw11111d1n"
                },
                "resume": {
                    "enabled": true,
                    "jwtSecretKey": "{random UUID}"
                }
            },
            "adaptation": {
                "bitrates": {
                    "minSelectableAudioBitrate": "100bps",
                    "maxSelectableAudioBitrate": "1000bps",
                    "minSelectableVideoBitrate": "100bps",
                    "maxSelectableVideoBitrate": "100bps"
                },
                "limitToPlayerSize": false,
                "exclude": false,
                "desktop": {
                    "bitrates": {
                        "minSelectableAudioBitrate": "100bps",
                        "maxSelectableAudioBitrate": "100bps",
                        "minSelectableVideoBitrate": "100bps",
                        "maxSelectableVideoBitrate": "100bps"
                    },
                    "qualityStabilityBalance": 1,
                    "limitToPlayerSize": false,
                    "exclude": false
                },
                "mobile": {
                    "bitrates": {
                        "minSelectableAudioBitrate": "100bps",
                        "maxSelectableAudioBitrate": "100bps",
                        "minSelectableVideoBitrate": "100bps",
                        "maxSelectableVideoBitrate": "100bps"
                    },
                    "qualityStabilityBalance": 1,
                    "limitToPlayerSize": false,
                    "exclude": false
                }
            },
            "buffer": {
                "audio": {
                    "backwardduration": 10,
                    "forwardduration": 10
                },
                "video": {
                    "backwardduration": 10,
                    "forwardduration": 10
                }
            }
        }
    },
    "defaults": {
        "auth": "cloudpay",
        "player": "bitmovin",
        "platform": "kaltura"
    },
    "entitlements": {
        "defaultEntryEntitlement": [
            "{CloudMatrix-Configuration}"
        ]
    },
}
```
Note: to apply this to fusion, you can use the exact same `adaptation` object in the `Mandatory Fusion Config`

## Playback - Ads
```json
{
    "description": "Cloudpay Ads",
    "tenantId": "{Unique-CloudMatrix}",
    "tags": [
        "A Tag"
    ],
    "auth": {
        "cloudpay": {
            "site": "{siteName}-payments.streamamg.com"
        }
    },
    "domains": [
        "https://playback-sdk-demo.dev.streamamg.com",
        "https://playback-sdk-demo.qa.streamamg.com",
        "https://playback-sdk-demo.staging.streamamg.com",
        "https://playback-sdk-demo.streamamg.com"
    ],
    "deliveryConfiguration": {
        "tokenizer": {
            "streamamg-test.akamaized.net": {
                "key": "{secure token by video team}",
                "tokenName": "hdnts",
                "type": "akamai",
                "windowSeconds": 60
            }
        }
    },
    "entitlementsUrl": "https://{CloudMatrixURL}/api/v1/entryentitlements/{entryid}",
    "platform": {
        "kaltura": {
            "partnerId": "123456",
            "partnerAdminSecret": "123456789123456789123"
        }
    },
    "player": {
        "bitmovin": {
            "license": "11111111-2222-3333-4444-555555555555",
            "integrations": {
                "mux": {
                    "player_name": "bitmovin-{name}",
                    "env_key": "1235aavjadwdw11111d1n"
                },
                "resume": {
                    "enabled": true,
                    "jwtSecretKey": "{random UUID}"
                }
            }
        }
    },
    "globalAdverts": [
        {
            "adType": "vast",
            "discardAfterPlayback": true,
            "id": "some-id",
            "persistent": true,
            "position": "50%",
            "preloadOffset": 0,
            "skippableAfter": 0,
            "url": "url"
        }
    ],
    "defaults": {
        "auth": "cloudpay",
        "player": "bitmovin",
        "platform": "kaltura"
    },
    "entitlements": {
        "defaultEntryEntitlement": [
            "{CloudMatrix-Configuration}"
        ]
    },
}
```
Note: to apply this to fusion, you can use the exact same `globalAdverts` object in the `Mandatory Fusion Config`
