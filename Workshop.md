

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

Demonstroidaanko ssh-keygen -t rsa, cat .ssh/id_rsa.pub >> .ssh/authorized_keys (plus
tarkista tiedosto-oikeudet) ssh localhost ja sit ilman localia?



Hello world!
------------

   * tehdään helloworld.yml

---

- hosts: dev
  tasks: 
    - debug: msg="hello world!"


lisätään:

  vars:
    message: "helo world-o"

    - debug: msg="{{ message }}"

(quotet koska ei haluta että yaml tulkitsee sen yaml dictiksi.)


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

aja: ansible-playbook helloworld.yml

=> ERROR: found role but did not find ..

edit roles/helloworld-server/tasks/main.yml:

---

- name: debug
  debug: msg="{{ message }}"






