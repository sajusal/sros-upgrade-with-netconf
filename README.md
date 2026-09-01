# Nokia SR OS Upgrade Using NETCONF

This tutorial is on using NETCONF to upgrade Nokia SR OS.

SR OS support for NETCONF and supported RPCs are available [here](https://documentation.nokia.com/sr/26-7/7x50-shared/system-management/netconf-intro.html)

SR OS Yang models are available [here](https://yangbrowser.nokia.com/sros/26.7.R1?from=0)

Test Device: 7750 SR-1x-92S

Current Software version: 26.3.R3

Target Software version: 26.7.R1

## NETCONF configuration on SR OS

Enable NETCONF listen mode. Default port is 830.
Enable config auto save when commit is done via NETCONF

```xml
/configure system management-interface netconf auto-config-save true
/configure system management-interface netconf listen admin-state enable
/configure system management-interface netconf listen port 830
```

Enable NETCONF RPCs under user profile:

Example is for `administrative` profile which is default.

```xml
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization action true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization cancel-commit true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization close-session true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization commit true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization copy-config true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization create-subscription true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization delete-config true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization discard-changes true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization edit-config true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization get true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization get-config true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization get-data true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization get-schema true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization kill-session true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization lock true
/configure system security aaa local-profiles profile "administrative" netconf base-op-authorization validate true
```

Assign profile to user and enable NETCONF access for user.

Example is for `admin` user.

```xml
/configure system security user-params local-user user "admin" access netconf true
/configure system security user-params local-user user "admin" console member ["administrative"]
```

## Connecting to SR OS using NETCONF

From a linux host, run:

```xml
ssh -p 830 username@ip-address -s netconf
```

Upon receiving this request, SR OS will respond with list of capabilities in a Hello RPC.

Before sending further RPCs, the user should respond back with a Hello.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<hello xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
<capabilities>
<capability>urn:ietf:params:netconf:base:1.0</capability>
</capabilities>
</hello>
]]>]]>
```

## Software Upgrade using NETCONF

### Verify current running version

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get>
    <filter type="subtree">
      <state xmlns="urn:nokia.com:sros:ns:yang:sr:state">
        <system>
          <version>
            <version-string/>
          </version>
        </system>
      </state>
    </filter>
  </get>
</rpc>
]]>]]>
```

Expected output:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <data>
        <state xmlns="urn:nokia.com:sros:ns:yang:sr:state">
            <system>
                <version>
                    <version-string>TiMOS-C-26.3.R3 cpm/x86hops64 Nokia 7750 SR-1x-92S Copyright (c) 2000-2026 Nokia.\nAll rights reserved. All use subject to applicable license agreements.\nBuilt on Wed May 27 21:47:33 UTC 2026 by builder in /builds/263B/R3/panos</version-string>
                </version>
            </system>
        </state>
    </data>
</rpc-reply>
```

### Create a new directory

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <action xmlns="urn:ietf:params:xml:ns:yang:1">
        <file xmlns="urn:nokia.com:sros:ns:yang:sr:oper-file">
            <make-directory>
                <url>cf3:\\TiMOS-SR-26.7.R1</url>
            </make-directory>
         </file>
    </action>
</rpc>
]]>]]>
```

Expected output:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nokiaoper="urn:nokia.com:sros:ns:yang:sr:oper-file">
    <nokiaoper:operation-id>9</nokiaoper:operation-id>
    <nokiaoper:start-time>2020-12-06T07:21:55.8Z</nokiaoper:start-time>
    <nokiaoper:status>completed</nokiaoper:status>
    <nokiaoper:end-time>2020-12-06T07:21:55.8Z</nokiaoper:end-time>
</rpc-reply>
```

### Transfer software image

Transfer the new software image files to the newly created directory using methods like SFTP.
This is outside the context of NETCONF.

### Verify checksum of the image files

Note: checksum calculation takes a few seconds.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <action xmlns="urn:ietf:params:xml:ns:yang:1">
    <file xmlns="urn:nokia.com:sros:ns:yang:sr:oper-file">
        <checksum>
            <url>cf3://TiMOS-SR-26.7.R1/cpm.tim</url>
            <type>sha256</type>
        </checksum>
    </file>
  </action>
</rpc>
]]>]]>
```

Expected output:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nokiaoper="urn:nokia.com:sros:ns:yang:sr:oper-file">
    <nokiaoper:operation-id>10</nokiaoper:operation-id>
    <nokiaoper:start-time>2020-12-06T07:23:47.2Z</nokiaoper:start-time>
    <nokiaoper:results>
        <nokiaoper:checksum-value>3e825c17375683f936aea5cdc6e6b1ce50000000000000000000009e1c165b2</nokiaoper:checksum-value>
    </nokiaoper:results>
    <nokiaoper:status>completed</nokiaoper:status>
    <nokiaoper:end-time>2020-12-06T07:24:14.1Z</nokiaoper:end-time>
</rpc-reply>
```

Repeat this step for all image files.

### Verify version of the image files

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <action xmlns="urn:ietf:params:xml:ns:yang:1">
    <file xmlns="urn:nokia.com:sros:ns:yang:sr:oper-file">
        <version>
            <url>cf3://TiMOS-SR-26.7.R1/cpm.tim</url>
        </version>
    </file>
  </action>
</rpc>
]]>]]>
```

Expected output:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nokiaoper="urn:nokia.com:sros:ns:yang:sr:oper-file">
    <nokiaoper:operation-id>11</nokiaoper:operation-id>
    <nokiaoper:start-time>2020-12-06T07:25:13.3Z</nokiaoper:start-time>
    <nokiaoper:results>
        <nokiaoper:version>
            <nokiaoper:version-number>C-26.7.R1</nokiaoper:version-number>
            <nokiaoper:version-string>Fri Jul 10 17:42:07 UTC 2026 by builder in /builds/267B/R1/panos</nokiaoper:version-string>
        </nokiaoper:version>
    </nokiaoper:results>
    <nokiaoper:status>completed</nokiaoper:status>
    <nokiaoper:end-time>2020-12-06T07:25:13.3Z</nokiaoper:end-time>
</rpc-reply>
```

### Copy boot.ldr to root directory

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <action xmlns="urn:ietf:params:xml:ns:yang:1">
        <file xmlns="urn:nokia.com:sros:ns:yang:sr:oper-file">
            <copy>
                <source-url>cf3:\\TiMOS-SR-26.7.R1\boot.ldr</source-url>
                <destination-url>cf3:\\boot.ldr</destination-url>
                <force/>
            </copy>
         </file>
    </action>
</rpc>
]]>]]>
```

Expected output:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nokiaoper="urn:nokia.com:sros:ns:yang:sr:oper-file">
    <nokiaoper:operation-id>15</nokiaoper:operation-id>
    <nokiaoper:start-time>2020-12-06T07:29:20.3Z</nokiaoper:start-time>
    <nokiaoper:status>completed</nokiaoper:status>
    <nokiaoper:end-time>2020-12-06T07:29:21.6Z</nokiaoper:end-time>
</rpc-reply>
```

Verify version of boot.ldr in root directory

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <action xmlns="urn:ietf:params:xml:ns:yang:1">
    <file xmlns="urn:nokia.com:sros:ns:yang:sr:oper-file">
        <version>
            <url>cf3://boot.ldr</url>
        </version>
    </file>
  </action>
</rpc>
]]>]]>
```

Expected output:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nokiaoper="urn:nokia.com:sros:ns:yang:sr:oper-file">
    <nokiaoper:operation-id>16</nokiaoper:operation-id>
    <nokiaoper:start-time>2020-12-06T07:31:02.8Z</nokiaoper:start-time>
    <nokiaoper:results>
        <nokiaoper:version>
            <nokiaoper:version-number>L-26.7.R1</nokiaoper:version-number>
            <nokiaoper:version-string>Fri Jul 10 17:42:07 UTC 2026 by builder in /builds/267B/R1/panos</nokiaoper:version-string>
        </nokiaoper:version>
    </nokiaoper:results>
    <nokiaoper:status>completed</nokiaoper:status>
    <nokiaoper:end-time>2020-12-06T07:31:03.2Z</nokiaoper:end-time>
</rpc-reply>
```

### Synchronize CPMs

On systems with redundant CPMs, synchronize the boot environment.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <action xmlns="urn:ietf:params:xml:ns:yang:1">
    <admin xmlns="urn:nokia.com:sros:ns:yang:sr:oper-admin">
        <redundancy>
            <synchronize>
                <boot-environment/>
            </synchronize>
        </redundancy>
    </admin>
  </action>
</rpc>
]]>]]>
```

### Edit the BOF

Lock the BOF for editing.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <lock>
    <target>
      <configuration-region xmlns="urn:nokia.com:sros:ns:yang:sr:ietf-netconf-augments">
        bof
      </configuration-region>
      <candidate/>
    </target>
  </lock>
</rpc>
]]>]]>
```

Expected output:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <ok/>
</rpc-reply>
```

Edit the BOF:

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <edit-config>
    <target>
      <configuration-region xmlns="urn:nokia.com:sros:ns:yang:sr:ietf-netconf-augments">
        bof
      </configuration-region>
      <candidate/>
    </target>
    <config>
      <bof xmlns="urn:nokia.com:sros:ns:yang:sr:bof-conf">
        <image>
          <primary-location>cf3:\TiMOS-SR-26.7.R1</primary-location>
          <secondary-location>cf3:\TiMOS-SR-26.3.R3</secondary-location>
        </image>
      </bof>
    </config>
  </edit-config>  
</rpc>
]]>]]>
```

Expected output:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <ok/>
</rpc-reply>
```

Commit BOF changes:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <commit> 
    <configuration-region xmlns="urn:nokia.com:sros:ns:yang:sr:ietf-netconf-augments">
      bof
    </configuration-region>
  </commit>
</rpc>
]]>]]>
```

Expected output:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <ok/>
</rpc-reply>
```

Verify BOF config using GET:

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get-config>
    <source>
      <configuration-region xmlns="urn:nokia.com:sros:ns:yang:sr:ietf-netconf-augments">
        bof
      </configuration-region>
      <running/>
    </source>
    <filter>
      <bof xmlns="urn:nokia.com:sros:ns:yang:sr:bof-conf"/>
    </filter>
  </get-config>
</rpc>
]]>]]>
```

Unlock BOF:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <unlock>
    <target>
      <configuration-region xmlns="urn:nokia.com:sros:ns:yang:sr:ietf-netconf-augments">
        bof
      </configuration-region>
      <candidate/>
    </target>
  </unlock>
</rpc>
]]>]]>
```


Expected response:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <ok/>
</rpc-reply>
```

### Reboot the node

This completes the image transfer activities for the upgrade.

Before proceeding with the upgrade, ensure all other pre-upgrade activities like health checks, config backups are completed.

Reboot the node:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <action xmlns="urn:ietf:params:xml:ns:yang:1">
    <admin xmlns="urn:nokia.com:sros:ns:yang:sr:oper-admin">
      <reboot>
        <now/>
      </reboot>
    </admin>
  </action>
</rpc>
]]>]]>
```

If only the standby CPM needs to be rebooted:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <action xmlns="urn:ietf:params:xml:ns:yang:1">
    <admin xmlns="urn:nokia.com:sros:ns:yang:sr:oper-admin">
      <reboot>
        <card>standby</card>
        <now/>
      </reboot>
    </admin>
  </action>
</rpc>
]]>]]>
```

If a line card needs to be soft reset:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <action xmlns="urn:ietf:params:xml:ns:yang:1">
    <reset xmlns="urn:nokia.com:sros:ns:yang:sr:oper-reset">
      <card>
        <slot-number>1</slot-number>
        <reinitialize/>
      </card>
    </reset>
  </action>
</rpc>
]]>]]>
```

If a line card needs to be power cycled:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <action xmlns="urn:ietf:params:xml:ns:yang:1">
    <perform xmlns="urn:nokia.com:sros:ns:yang:sr:oper-perform">
      <card>
        <slot-number>1</slot-number>
        <power-cycle/>
      </card>
    </perform>
  </action>
</rpc>
]]>]]>
```

## Verify version post upgrade

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get>
    <filter type="subtree">
      <state xmlns="urn:nokia.com:sros:ns:yang:sr:state">
        <system>
          <version>
            <version-string/>
          </version>
        </system>
      </state>
    </filter>
  </get>
</rpc>
]]>]]>
```

Expected output:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <data>
        <state xmlns="urn:nokia.com:sros:ns:yang:sr:state">
            <system>
                <version>
                    <version-string>TiMOS-C-26.7.R1 cpm/x86hops64 Nokia 7750 SR-1x-92S Copyright (c) 2000-2026 Nokia.\nAll rights reserved. All use subject to applicable license agreements.\nBuilt on Fri Jul 10 17:42:07 UTC 2026 by builder in /builds/267B/R1/panos</version-string>
                </version>
            </system>
        </state>
    </data>
</rpc-reply>
```

This confirms the node was upgraded to SR OS 26.7.R1

## Disconnect NETCONF session

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <close-session/>
</rpc>
]]>]]>
```
