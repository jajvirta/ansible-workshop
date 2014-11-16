

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

(Idea: tehdään tuotanto/testiympäristöä vastaava setup tyhjään koneeseen pienellä vaivalla.)
 

Ansiblen inventory
------------------

   * oletuspaikka /etc/ansible/hosts, me tehdään lokaali tiedosto hosts
   * sinne: [dev]\nlocalhost 
   * testataan että toimii: ansible -i hosts -c local -m ping
      * normaalisti ajetaan ssh:lla kohdekoneille, local connection paljon nopeampi

TODO: Demonstroidaanko ssh-keygen -t rsa, cat .ssh/id_rsa.pub >> .ssh/authorized_keys (plus
tarkista tiedosto-oikeudet) ssh localhost ja sit ilman localia?



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


Helloworld-rooli
----------------


    mkdir -p roles/helloworld-server/tasks
    
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

=> changed: [localhost] => syntyy hakemisto

re-run

=> sanoo vain että ok












