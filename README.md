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
