# Auditd Configuration with Syslog-NG

## 1-) Debian Based Dist. Auditd Configuration with Syslog-NG

### 1.1 Installations

    sudo apt update -y
    sudo apt install -y auditd audispd-plugins
    sudo apt-get install syslog-ng syslog-ng-core

### 1.2 Disable Default

    systemctl stop rsyslog
    systemctl disable rsyslog

### 1.3 Auditd Configuration

vi /etc/audit/rules.d/audit.rules

    ## First rule - delete all
    -D

    ## Increase the buffers to survive stress events.
    ## Make this bigger for busy systems
    -b 8192

    ## Set failure mode to syslog
    -f 1

    -a always,exit -F arch=b64 -F euid=0 -F auid!=4294967295 -F perm=wxa -k Root_Action
    -a always,exit -F arch=b32 -F euid=0 -F auid!=4294967295 -F perm=wxa -k Root_Action
    -a always,exit -F arch=b64 -F euid!=0 -F auid!=4294967295 -F perm=wxa -k User_Action
    -a always,exit -F arch=b32 -F euid!=0 -F auid!=4294967295 -F perm=wxa -k User_Action

-----------------------------------------------------------------------------------

vi /etc/audit/auditd.conf // It gives an error in old Debian versions, skip it.

change log_format "log_format = ENRICHED" instead of "log_format = RAW"

-----------------------------------------------------------------------------------

nano /etc/audit/plugins.d/syslog.conf

change status, "active = yes" instead of "active = no"

### 1.4 SYSLOG-NG CONF Sample

nano /etc/syslog-ng/conf.d/auditd.conf

    source s_audit {
            file("/var/log/messages");
    };
    destination d_dest {
            network (
                    "1.1.1.1"
                    transport("tcp")
                    port(514)
                    );
    };

    log {
            source(s_audit);
            destination(d_dest);
    };

### 1.5 Start

    service ufw stop
    systemctl disable ufw
    systemctl restart auditd
    systemctl enable auditd
    systemctl restart syslog-ng
    systemctl enable syslog-ng

## 2-) RHEL/Centos Based Dist. Auditd Configuration with Syslog-NG

### 2.1 Installations

    sudo yum update -y
    sudo yum install syslog-ng syslog-ng-core -y

### 2.2 Disable Default

    systemctl stop rsyslog
    systemctl disable rsyslog

### 2.3 Auditd Configuration

vi /etc/audit/rules.d/audit.rules

    ## First rule - delete all
    -D

    ## Increase the buffers to survive stress events.
    ## Make this bigger for busy systems
    -b 8192

    ## Set failure mode to syslog
    -f 1

    -a always,exit -F arch=b64 -F euid=0 -F auid!=4294967295 -F perm=wxa -k Root_Action
    -a always,exit -F arch=b32 -F euid=0 -F auid!=4294967295 -F perm=wxa -k Root_Action
    -a always,exit -F arch=b64 -F euid!=0 -F auid!=4294967295 -F perm=wxa -k User_Action
    -a always,exit -F arch=b32 -F euid!=0 -F auid!=4294967295 -F perm=wxa -k User_Action

-----------------------------------------------------------------------------------

vi /etc/audit/auditd.conf

change log_format "log_format = ENRICHED" instead of "log_format = RAW"

-----------------------------------------------------------------------------------

vi /etc/audisp/plugins.d/syslog.conf

change status, "active = yes" instead of "active = no"

### 2.4 SYSLOG-NG CONF Sample

vi /etc/syslog-ng/conf.d/auditd.conf

    source s_audit {
            file("/var/log/messages");
    };
    destination d_dest {
            network (
                    "1.1.1.1"
                    transport("tcp")
                    port(514)
                    );
    };

    log {
            source(s_audit);
            destination(d_dest);
    };

### 2.5 Start

    service firewalld stop
    systemctl disable firewalld
    service auditd restart
    systemctl enable auditd
    systemctl restart syslog-ng
    systemctl enable syslog-ng
