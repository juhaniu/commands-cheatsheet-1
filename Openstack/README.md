### Get server list (name, IP, status)
openstack server list |  awk '{print $4, $9, $11, $6}'
