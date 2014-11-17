

serve.sh
--------
#!/bin/bash

set -e

if [[ $1 == "start" ]]; then
  cd /usr/local/hw/app/
  echo "starting hw server $(date +%H:%M) at port $(cat port.txt) " >> /usr/local/hw/hw.log
  pwd >> /usr/local/hw/hw.log
  python -m SimpleHTTPServer $(cat port.txt) >> /usr/local/hw/server.log 2>&1 &
  echo "started the server" >> /usr/local/hw/hw.log
elif [[ $1 == "stop" ]]; then
  kill $(pgrep -f SimpleHTTPServer)
fi

app_dir: /usr/local/hw/app
port_number: 7999


Serviceksi muuttaminen
----------------------

    - file: dest=/usr/local/hw state=directory
    - file: dest={{ app_dir }} state=directory
    
    - copy: src={{ item }} dest=/usr/local/hw/ mode=0755
      with_items:
        - serve.sh
    
    - file: src=/usr/local/hw/serve.sh dest=/etc/init.d/serve.sh state=link
    
    - template: src=port.txt.j2 dest=/usr/local/hw/app/port.txt
    
    - service: name=serve.sh state=restarted
    
    - name: deplaa appi
      copy: src=hw.txt dest={{ app_dir }}
    
    - name: poista vanha output
      file: dest=/usr/local/hw/output.txt state=absent
    
    - name: smoketestaa palvelu
      get_url: url=http://localhost:{{ port_number }}/hw.txt dest=/usr/local/hw/output.txt
      register: smoke_result
    
    #- debug: var=smoke_result
    
    
    #- name: veppiservaa
    #  command: /bin/bash /usr/local/hw/serve.sh
    #  async: 10
    #  poll: 0
    
    
