
Aloitus
-------

   * vagrant up
   * vagrant ssh
   * ansible --version
   * cd /vagrant
      * jaettu kansio hostin ja guestin välillä
      * synced_folder -rimpsu Vagrantfilessä on sen takia, että Ansible ei suostu käyttämään executablea inventorya
      * jaettu kansio => voi tehdä jutut virtuaalikoneen ulkopuolella
      * (voi editoida kummalla puolella vaan)


Jos tulee mount-ongelmia:

    sudo ln -s /opt/VBoxGuestAdditions-4.3.10/lib/VBoxGuestAdditions /usr/lib/VBoxGuestAdditions

Ansiblen inventory
------------------

   * oletuspaikka `/etc/ansible/hosts`, me tehdään lokaali tiedosto hosts
   * sinne: [dev]\nlocalhost 
   * testataan että toimii: ansible -i hosts -c local all -m ping
      * normaalisti ajetaan ssh:lla kohdekoneille, local connection paljon nopeampi

Normaalisti tehtäis: ssh-keygen -t rsa, cat .ssh/id_rsa.pub >> .ssh/authorized_keys (plus
tarkista tiedosto-oikeudet) ssh localhost ja sit ilman localia.

Hello world!
------------

tehdään `helloworld.yml`

    ---

    - hosts: dev
      tasks: 
        - debug: msg="hello world!"


lisätään:

  vars:
    name: jarno

    - debug: msg="hello world, {{ name }}!"


lisätään:

- hosts: dev
  connection: local <<==

lisätään myös ansible.cfg, jossa:

[defaults]
hostfile=hosts

=> päästään eroon komentorivioptioista kokonaan

lisätään vielä toinen host (1.2.3.4)? ja esitellään --limit=localhost?


-v -vv -vvv

käydään läpi tuloste:

GATHERING FACTS (gather_facts: no)

ansible -i hosts -c local -m setup dev

gather_facts: no => voi ampua itseään jalkaan!

Helloworld-rooli
----------------

    mkdir -p roles/helloworld-server/tasks

helloworld.yml:
    roles:
      - helloworld-server

    ansible-playbook helloworld.yml
    
=> ERROR: found role but did not find ..

editoidaan roles/helloworld-server/tasks/main.yml:

    ---
    
    - name: debug
      debug: msg="hello world {{ name }}" 

Jotain hyödyllisempää
---------------------

    - name: perushakemisto
      file: dest=/usr/local/hw state=directory

run

no can do! lisätään sudo

=> changed: [localhost] => syntyy hakemisto

re-run

=> sanoo vain että ok


ei varmaankaan haluta että käyttäjä on root-juures eli tehdään
uusi käyttäjä


    - name: tee käyttäjä
      user: name=veijo shell=/bin/bash

re-run

    sudo su - veijo

korjataan hakemiston luontia:

    file: dest=/usr/local/hw state=directory *owner=veijo*

re-run

    ls -lrt /usr/local/ | grep hw
         

ryhmille löytyy vastaava

[n. 10 minuuttia tähän asti]

Koodataan "serveri"
-------------------

komentorivillä: 

    python -m SimpleHTTPServer &
    wget -O- localhost:8000
    fg
    C-c

serve.sh:

    #/bin/bash
    set -e
    if [[ $1 == "start" ]]; then
      cd /usr/local/hw/
      python -m SimpleHTTPServer >> /usr/local/hw/server.log 2>&1 &
    elif [[ $1 == "stop" ]]; then
      kill $(pgrep -f SimpleHTTPServer)
    fi

    - name: kopsaa serveri
      copy: src=serve.sh dest=/usr/local/hw mode=0755

huomatkaa, että copy toimii myös sinne remote-hosteille tällai kivasti



tehdään siitä "palvelu":

    - file: src=/usr/local/hw/serve.sh dest=/etc/init.d/serve.sh state=link

joten voidaan komentaa sitä niin kuin palvelua:

    - service: name=serve.sh state=restarted

("restarted" koska ei jakseta tehdä myös "status" funkkaria)

re-runataan 

=> success?
    
    - name: smoketestaa palvelu
      get_url: url=http://localhost:8000 dest=/usr/local/hw/output.txt

run
re-run => ok?
`get_url` ei runaa uudelleen kun dest on jo olemassa
    
    - name: poista smoketestin output
      file: dest=/usr/local/hw/output.txt state=absent

"dirty trick":

    - name: smoketestaa palvelu
      get_url: url=http://localhost:8000/hw.txt dest=/usr/local/hw/

tai:

    - name: smoketestaa palvelu
      get_url: url=http://localhost:8000/hw.txt force=yes dest=/usr/local/hw/

ladataan aina uudelleen. Pointtina myös se, että jos oikeasti haetaan
jotain isoa get_urlilla, niin sitä ei haeta aina uudelleen!

testataan, että smoketesti toimii:

    - service: name=serve.sh state=stopped

=>

    TASK: [hw-server | smoketestaa palvelu] ***************************************
    failed: [localhost] => {"failed": true, "item": "", "status_code": -1}
    msg: Request failed: <urlopen error [Errno 111] Connection refused>
    
    FATAL: all hosts have already failed -- aborting

(tai laitetaan kommenttiin restart ja suljetaan käsin se palvelu, niin kosahtaa)



    - template: src=port.txt.j2 dest=/usr/local/hw/port.txt

    python -m SimpleHTTPServer $(cat port.txt)


host_vars:
    port_number: 7999




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





