




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






ansible -komento 
