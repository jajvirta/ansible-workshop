

serve.sh
--------


#!/bin/bash

set -e

if [[ $1 == "start" ]]; then
  echo "starting hw server $(date +%H:%M)" >> /usr/local/hw/hw.log
  cd /usr/local/hw/app/
  pwd >> /usr/local/hw/hw.log
  python -m SimpleHTTPServer >> /usr/local/hw/server.log 2>&1 &
  echo "started the server" >> /usr/local/hw/hw.log
elif [[ $1 == "stop" ]]; then
  kill $(pgrep -f SimpleHTTPServer)
fi


Serviceksi muuttaminen
----------------------



    - file: dest=/usr/local/hw state=directory
    - file: dest={{ app_dir }} state=directory
    
    - copy: src={{ item }} dest=/usr/local/hw/ mode=0755
      with_items:
        - serve.sh
    
    - file: src=/usr/local/hw/serve.sh dest=/etc/init.d/serve.sh state=link
    
    - service: name=serve.sh state=started
    
    - name: deplaa appi
      copy: src=hw.txt dest={{ app_dir }}
    
    - name: smoketestaa palvelu
      get_url: url=http://localhost:8000/hw.txt dest=/usr/local/hw/output.txt



