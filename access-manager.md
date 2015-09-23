# PubNub Access Manager (PAM)

PubNub Access Manager (PAM) provides fine-grained access controls to PubNub
Data Streams and DataSync Objects. It presents a minimal REST API for secure
administration tasks, and transparently protects Data Stream and DataSync
resource access. This document divides these two responsibilities clearly into
their own sections; REST API, and ACL Enforcement.

## General Information

* Global Permissions
    Do Not Provide ```auth``` nor ```channel``` in the <sign> query-string parameters
    
* Channel Permissions
    Provide only a ```channel``` parameter, but not ```auth``` parameter, this then sets permissions on the channel itself
     overriding any auth-key permissions
     
* AuthKey Permissions
    Provide both ```auth``` and ```channel``` parameters to set the permissions for one or more channels and one or more authkeys
    
Both ```auth``` and ```channel``` accept comma-separated lists.


## Request Format

    HTTP GET
     
    https://pubsub.pubnub.com/v1/auth/grant/sub-key/{subscribe-key}?signature=<sign>&auth={auth-key}
    
    

#### Computing the Signature for Request - <sign> QS Param

`<sign>` is computed using ```HMAC+SHA256``` with the user's ```secret key``` as the 
signing key, and the request string as the message. The request string is composed of
the request query parameters concatenated to the subscribe key, publish key, and
method (`grant`, `revoke`, or `granted`) in the following format string:

    "{sub_key}\n{pub_key}\n{method}\n{query_string}"

Here is a full example message:

    {subscribe-key}
    {publish-key}
    grant
    auth={auth-key-to-be-updated}&channel=my_channel&r=1&w=1&timestamp=123456789&ttl=1440

##### Query String Required Formatting

* Query string parameters must be sorted lexicographically (case-sensitive) by
key. 

* Secondly, all characters in the query string parameters must be
percent-encoded *except* alphanumeric, hyphen, underscore, and period; E.g. all
characters matching the RegExp `/[^0-9a-zA-Z\-_\.]/`. 

* Space characters must be replaced by `%20` (NOT `+` character). 

* Each key-value pair must be separated by ampersand characters. 

* Unicode characters must be broken up into UTF-8 encoded bytes before percent-encoding.

* Final signed value should be Base64 encoded using the "URL safe" characters `-` and `_` replacing `+` and `/` respectively.

Here is an example of a query string containing unicode characters (PoundsSterling is just there to demonstrate encoding/sorting):

    auth={auth-key-to-be-updated}&r=1&w=1&ttl=60&timestamp=123456789&PoundsSterling=£13.37

And here is the same query string after ***sorting and percent-encoding*** (PoundsSterling is just there to demonstrate encoding/sorting):

    PoundsSterling=%C2%A313.37&auth=joker&r=1&timestamp=123456789&ttl=60&w=1

Let's imagine the demo account's secret key is:

    wMfbo9G0xVUG8yfTfYw5qIdfJkTd7A

The signature generated for this request is Base64 encoded using the "URL safe"
characters `-` and `_` replacing `+` and `/` respectively:

    v2rgQQ1eFzk8omugFV9V1_eKRUvvMv9jyC9Z-L1ogdw=

This signature is then percent-encoded according to standard query parameter
percent-encoding practices. E.g. the `=` character is transformed into `%3D`.

    v2rgQQ1eFzk8omugFV9V1_eKRUvvMv9jyC9Z-L1ogdw%3D

## Response

Depending on the parameters provided (channel name, auth

    {
        "status": 200,
        "message": "Success",
        "payload": {
            "ttl": 1440,
            "auths": {
                "password": {
                    "r": 1,
                    "w": 0
                }
            },
            "subscribe_key": "{subscribe-key}",
            "level": "user",
            "channel": "my_channel"
        },
        "service": "Access Manager"
    }
