# Requirements

1. Ubuntu 20.04+
2. OpenSSH installed and configured
3. CMake, GCC, pkg-config
4. Built from source: `libyang` `libnetconf2` `sysrepo` `netopeer2`

# Setup Instructions
1. **Clone & Build All Dependencies** : Refer to this [repo](https://github.com/KRIISHSHARMA/netopeer-setup)

2. **Make a Yang  model**  : For example this is from [wiki](https://en.wikipedia.org/wiki/YANG#Example)
```yang
module example-sports {

  namespace "http://example.com/example-sports";
  prefix sports;

  import ietf-yang-types { prefix yang; }

  typedef season {
    type string;
    description
      "The name of a sports season, including the type and the year, e.g,
       'Champions League 2014/2015'.";
  }

  container sports {
    config true;

    list person {
      key "name";
      leaf name { type string; }
      leaf birthday { type yang:date-and-time; mandatory true; }
    }

    list team {
      key "name";
      leaf name { type string; }
      list player {
        key "name season";
        unique number;
        leaf name { type leafref { path "/sports/person/name"; }  }
        leaf season { type season; }
        leaf number { type uint16; mandatory true; }
        leaf scores { type uint16; default 0; }
      }
    }
  }
}
```
- ***you can also use yanglint to check  validating above yang file*** 

3. **Install YANG Module into Sysrepo** : `sudo sysrepoctl -i example-sports.yang`

    - check correct installation : `sysrepoctl -l | grep example-sports`
      ```
      example-sports                |            | I     | root:root     | 600           | 600           |            |    
      ```
4. Add XML Config (Optional) : Add startup data for module using sysrepocfg (more on [datastores](https://netopeer.liberouter.org/doc/sysrepo/master/html/))

```xml
<sports xmlns="http://example.com/example-sports">
  <person>
    <name>Sachin Tendulkar</name>
    <birthday>1973-04-24T00:00:00Z</birthday>
  </person>
  <person>
    <name>Brian Lara</name>
    <birthday>1969-05-02T00:00:00Z</birthday>
  </person>
  <person>
    <name>Jacques Kallis</name>
    <birthday>1975-10-16T00:00:00Z</birthday>
  </person>
  <person>
    <name>Ricky Ponting</name>
    <birthday>1974-12-19T00:00:00Z</birthday>
  </person>

  <team>
    <name>India Legends</name>
    <player>
      <name>Sachin Tendulkar</name>
      <season>World Cup 2011</season>
      <number>10</number>
      <scores>482</scores>
    </player>
  </team>

  <team>
    <name>West Indies Legends</name>
    <player>
      <name>Brian Lara</name>
      <season>World Cup 2003</season>
      <number>9</number>
      <scores>592</scores>
    </player>
  </team>

  <team>
    <name>South Africa Legends</name>
    <player>
      <name>Jacques Kallis</name>
      <season>World Cup 2007</season>
      <number>3</number>
      <scores>489</scores>
    </player>
  </team>

  <team>
    <name>Australia Legends</name>
    <player>
      <name>Ricky Ponting</name>
      <season>World Cup 2007</season>
      <number>14</number>
      <scores>539</scores>
    </player>
  </team>
</sports>
```
- **Import this xml file configuration**,***Its format will be automatically detected based on the extension. If not applicable (or the data are read from STDIN), it can be manually determined.***
   - `sudo sysrepocfg --import=<path to >/sports.xml --datastore=startup` : [reference](https://netopeer.liberouter.org/doc/sysrepo/libyang1/html/sysrepocfg.html)
 
5. **Start Netopeer2 Server**
   - use given script to start netopeer server
   - check `/var/log/netopeer2-server.log` for any unexpected behavior or errors

