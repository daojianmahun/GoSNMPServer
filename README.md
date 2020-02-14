GoSNMPServer
======
[![Build Status](https://travis-ci.org/slayercat/GoSNMPServer.svg?branch=master)](https://travis-ci.org/slayercat/GoSNMPServer)
[![GoDoc](https://godoc.org/github.com/slayercat/GoSNMPServer?status.png)](https://godoc.org/github.com/slayercat/GoSNMPServer)

GoSNMPServer is an SNMP server library fully written in Go. It **WILL** provides Server Get,
GetNext, GetBulk, Walk, BulkWalk, Set and Traps. It supports IPv4 and
IPv6, using __SNMPv2c__ or __SNMPv3__. Builds are tested against
linux/amd64 and linux/386.

TL;DR
-----
Build your own SNMP Server, try this:
```shell
go install github.com/slayercat/GoSNMPServer/cmd/gosnmpserver
$(go env GOPATH)/bin/gosnmpserver run-server
snmpwalk -v 3 -l authPriv  -n public -u testuser   -a md5 -A testauth -x des -X testpriv 127.0.0.1:1161 1
```

Quick Start
-----
```golang
import "github.com/slayercat/gosnmp"
import "github.com/slayercat/GoSNMPServer"
import "github.com/slayercat/GoSNMPServer/mibImps/ucdMib"
```

```golang

master := GoSNMPServer.MasterAgent{
    Logger: GoSNMPServer.NewDefaultLogger(),
    SecurityConfig: GoSNMPServer.SecurityConfig{
        AuthoritativeEngineBoots: 1,
        Users: []gosnmp.UsmSecurityParameters{
            {
                UserName:                 c.String("v3Username"),
                AuthenticationProtocol:   gosnmp.MD5,
                PrivacyProtocol:          gosnmp.DES,
                AuthenticationPassphrase: c.String("v3AuthenticationPassphrase"),
                PrivacyPassphrase:        c.String("v3PrivacyPassphrase"),
            },
        },
    },
    SubAgents: []*GoSNMPServer.SubAgent{
        {
            CommunityIDs: []string{c.String("community")},
            OIDs:         ucdMib.All(),
        },
    },
}
server := GoSNMPServer.NewSNMPServer(master)
err := server.ListenUDP("udp", "127.0.0.1:1161")
if err != nil {
    logger.Errorf("Error in listen: %+v", err)
}
server.ServeForever()
```


Serve your own oids
-----
This library provides some common oid for use. See [ucdMib](https://github.com/slayercat/GoSNMPServer/tree/master/mibImps/ucdMib) for code, See [![GoDoc](https://godoc.org/github.com/slayercat/GoSNMPServe/mibImps/ucdMibr?status.png)](https://godoc.org/github.com/slayercat/GoSNMPServer/mibImps/ucdMib) here.


Append `GoSNMPServer.PDUValueControlItem` to your SubAgent OIDS:
```golang
{
    OID:      fmt.Sprintf("1.3.6.1.2.1.2.2.1.1.%d", ifIndex),
    Type:     gosnmp.Integer,
    OnGet:    func() (value interface{}, err error) { return GoSNMPServer.Asn1IntegerWrap(ifIndex), nil },
    Document: "ifIndex",
},
```
Supports Types:
- Boolean
- Integer
- BitString
- OctetString
- ObjectIdentifier
- IPAddress
- Counter32
- Gauge32
- TimeTicks
- Counter64
- Uinteger32
- OpaqueFloat
- OpaqueDouble

Could use wrap function for detect type error. See `GoSNMPServer.Asn1IntegerWrap` / `GoSNMPServer.Asn1IntegerUnwrap` and so on.

Thanks
-----
This library is based on **[soniah/gosnmp](https://github.com/soniah/gosnmp)** for encoder / decoders. (made a [fork](https://github.com/slayercat/gosnmp) for maintenance)
