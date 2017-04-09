# pFsense
Updates for UDP packet loss bewteen containers
Tested with pFsense 2.2.6, SIEMonster 2.5

Upgrade the Syslog-NG container in the Rancher UI, change the Network to Host option and check 'Enable Rancher DNS service discovery'.

Execute a shell to the Syslog-NG container from an SSH terminal session.
Backup the file /etc/syslog-ng/syslog-ng.conf and replace as follows:
```
wget -O /etc/syslog-ng/syslog-ng.conf https://raw.githubusercontent.com/siemonster/pfsense/master/syslog-ng.conf

```
Restart the Syslog-ng container.

Execute a shell to the Logstash container, remove the file /config-dir/20-pfsense-filter.conf and download new filter file.
```
wget -O /config-dir/22-pfsense-filter.conf https://raw.githubusercontent.com/siemonster/pfsense/master/22-pfsense-filter.conf

```
Download new pFsense patterns.
```
wget -O /etc/logstash/patterns/pfsense-patterns-2 https://raw.githubusercontent.com/siemonster/pfsense/master/pfsense-patterns-2

```
Modify the 03-multisyslog-filter.conf file to add pFsense tagging (see example 03-multisyslog-filter.example.conf in this repo).

```
#change to pfSense IP address
      if [syslog_host] =~ /10\.2\.2\.254/ {
      mutate {
        add_tag => ["PFSense"]
}
```
Modify the /config-dir/99-outputs.conf file, adding a pFsense output stanza.
```
if "PFSense" in [tags] {
    elasticsearch {
         hosts => ["es-client:9200"]
         index => "logstash-pfsense-%{+YYYY.MM.dd}"
    }
  }
```
Restart the Logstash container.

Create a new index in Kibana - logstash-pfsense-*

If required, modify the searches and visualisations in this repo to change the pFsense network interface. These examples use 'em0' as the WAN interface.

Import the searches, visualisations & dashboard into Kibana - Settings - Objects - Import.
