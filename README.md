Image collectd
==============

This Dockerfile will all the creating of a docker container running on a CoreOs host to monitor 
the CoreOS for use with RightScale.  

The sources of collectd are altered due to the hard path /proc. This allows monitoring of the CoreOS host.

How to run :
See the RightScript "CoreOS Monitoring" in the RightScale MultiCloud Library.  Add the script to the Boot sequence of the server template.

Requires RightLink 10

```
#!/bin/bash -ex
# see more detail about monitoring here
# http://docs.rightscale.com/rl10/reference/rl10_monitoring.html
# see RS collectd install script
# https://github.com/rightscale/rightlink_scripts/blob/master/rll/collectd.sh

RS_RLL_PORT=`cat /var/run/rightlink/secret | grep PORT | cut -d= -f2`

/home/rightlink/rsc rl10 put_hostname /rll/tss/hostname hostname=$COLLECTD_SERVER

#sudo /usr/bin/docker kill collectd
#sudo /usr/bin/docker rm collectd
sudo /usr/bin/docker pull rsservices/coreos-collectd:latest
sudo /usr/bin/docker  run -t -i --name collectd -d --net=host -v /proc:/proc_host:ro \
-e RS_RLL_PORT=$RS_RLL_PORT \
-e RS_INSTANCE_UUID=$RS_INSTANCE_UUID \
rsservices/coreos-collectd 

# Add the RightScale monitoring active tag
auth_tag=rs_monitoring:state=auth
/home/rightlink/rsc --rl10 cm15 multi_add /api/tags/multi_add resource_hrefs[]=$RS_SELF_HREF tags[]=$auth_tag &&\
  logger -s -t RightScale "Setting monitoring active tag"
  ```
