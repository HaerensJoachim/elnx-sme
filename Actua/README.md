# Opdracht Actualiteit - Speed Up Ansible Playbooks

## Inleiding

Als deel voor de eintaak van het vak Linux moest ik ook een actua opdracht kiezen. We hadden 2 opties zelf een bijdrage leveren aan een open source project of een tool toepasse, op onze hoofdopdracht.

Ik koos ervoor om een tool toe te passen op de mijn hoodfopdracht. Ik ga volgens het artikel van Johnson "Making Ansible a Bit Faster" (link in de bron), mijn ansible opstelling sneller proberen maken. In deze taak ga ik bekijken of zijn aanbevelingen die hij aantoonde klopte. Daarbij ook een optimale snelheid proberen te vinden voor mijn ansible opstelling.

In deze taak test ik alles telkens alleen op de server pr011. Ik voer ook altijd telkens een ``vagrant provision pr011`` uit  en geen ``vagrant up pr011 `` aangezien we enkel het asnible playbook testen. De test computer is een HP Envy 700-014eb. De specificaties en foto zijn te vinden hieronder.

| HP ENVY 700-014EB |                |
| :---           | :---           |
| **Processor**     | Intel Core i7 4770              | 		
| **Processorkernen** | Quad core (4)              | 
| **Kloksnelheid** | 3400MHz              | 
|**Turbo Frequentie** | 3900 MHz            | 
| **RAM-Geheugen**   | 16 GB | 
| **Type RAM-Geheugen**  | DIMM DDR3 | 
| **Opslagtype**  | SSD | 
| **Moederbord** | MS-7826 (Kaili)  |

*Tabel 1: Specificaties HP Envy 700-014eb*

<img src="https://github.com/MaartenDeS/elnx-sme/blob/soluation/Actua/Foto's/hp.png" width="500">

*Foto 1: HP Envy 700-014eb*


## Tijdsmeting Plugin

Het allereerste wat ik moet doen is een tijdsmeting plugin installeren zodat ik alles kan navolgen. Via het artikel van Johnson kwam ik op een Ansible plugin terecht. Deze plugin is gemaakt door jlafon en is voor Ansible 1.x en voor Ansible 2.0.

Ik clownde de repository en volgde de instructies. Er waren 2 verschillende mogelijkheden om deze plugin toe te voegen. De eerste was voor Ansible 2.0, de andere voor Ansible 1.x. 

Voor de manier van Ansible1.x  moest ik de map **callback_plugins** met daarin de file **profile_tasks.py** in de map naast mijn playbook. Deze map in mijn opstelling was de map Ansible. Toen ik hierna testte kon ik de tijdsmetingen zien.

De manier voor Ansible 2.0 leek werkte helemaal anders. Hierin moest ik in de **ansible.cfg** file de lijn `` pipelining = True`` zetten op zich gee probleem. De opstelling gebruikte de default locatie van deze file *etc/ansible/ansible.cfg*. Deze kan je overriden door in je home directory ook een **ansible.cfg** file te zetten. In de opstelling is de home directory */home/vagrant/*. Eens ik deze had gevonden maakte ik in Ansible een pre_task aan zodat er elke keer de machine opstart een **ansible.cfg** file wordt toegevoegd in die map. In deze file stond de juiste setting dus wel aan.


## Snelheid Normale Ansible

Nu kan ik de standaardtijden meten van Ansible. Ik doe 10 tijdsmetingen zodat ik een goed beeld heb van wat de normale tijd ongeveer is. Hieronder zijn de  de tijden een tabel gegooid voor een duidelijk overzicht. Ook ziet u een foto van hoe de weergave er uitzag


![voorbeeld](https://github.com/MaartenDeS/elnx-sme/blob/soluation/Actua/Foto's/voorbeeld.png)
*Foto 2: voorbeeld tijdsweergave*



| Tijdsmeting |Tijd (Plugin weergave - Seconden)               |
| :---           | :---           |
| Tijdsmeting 1  | 0:00:20.720 - 20.720 seconden     | 		
|Tijdsmeting 2 | 0:00:20,923 - 20,923 seconden         | 
| Tijdsmeting 3 |0:00:20,768 - 20,768 seconden        | 
|Tijdsmeting 4 | 0:00:20,969 - 20,969 seconden     | 
| Tijdsmeting 5| 0:00:22,731 - 22,731 seconden    | 
| Tijdsmeting 6  | 0:00:20,915 - 20,915 seconden     | 		
|Tijdsmeting 7 | 0:00:20,909 - 20,909 seconden         | 
| Tijdsmeting 8 |0:00:20,902 - 20,902 seconden        | 
|Tijdsmeting 9 | 0:00:20,897 - 20,897 seconden     | 
| Tijdsmeting 10| 0:00:21,059 - 21,059 seconden    | 

*Tabel 2: Tijdsweergavenormale ansible*

Hieruit kunnen we afleiden dat de gemiddelde normale tijd  21,079 seconden is, de mediaan is 20,912 seconden.



## SSH pipelining

SSH pipelining is volgens Johnson een makkelijke manier om Ansible te versnellen. Het is een ansible feature om het aantal verbindingen naar een host te verminderen. Ansible zou normaal een tijdelijke *~/.ansbible* (via SSH). Dan zulllen ze voor elke task de module kopiëren naar de directory met sftp of scp en dan deze module uitvoeren (weer met SSH). Als pipelining aanstaat zal Ansible maar 1 keer per taak connecteren met ssh namelijk met het uitvoeren met python. de module zal worden geschreven naar de stdin. Om deze functie aan te zetten heb je 2 mogelijkheden. Je kan dit doen via jouw **playbook** of via het **ansible.cfg** bestand.

Voor de manier met het **ansible.cfg** bestand gaan we naar het ansible.cfg bestand dat we kopiëren naar onze home directory. In dat bestand voegen we de regel ``pipelining = True`` toe onder de regel `` [ssh_connection]``.

De tweede manier met het **playbook** is een beetje anders. Daarbij voegen we bij ``- hosts: pr011`` eerst de regel ``vars: `` toe. Dan onder die regel met de juiste syntax voegen we de regel ``ansible_ssh_pipelining: yes``toe.

Ik heb de manier in het **ansible.cfg** bestand toegpast. Om te bekijken of pipelining aanstaat gebruiken we het commando `` ansible localhost -vvv -m shell -a 'echo ok' ``. Als pipelining aanstaat krijg ik deze output(zie foto 3). Als pipeling uitstaat krijg je een veel langere output(zie foto 4). Dan roept hij 3 tot 5 keer het **command.py** bestand op. Op de foto duidelijk maar 1 keer en daaruit blijkt dat pipelining aanstaat.

![pipeaan](https://github.com/MaartenDeS/elnx-sme/blob/soluation/Actua/Foto's/pipeaan.png)
*Foto 3: SSH pipelining output - Pipelining aan*

![pipeuit](https://github.com/MaartenDeS/elnx-sme/blob/soluation/Actua/Foto's/pipeuit.png)
*Foto 4: SSH pipelining output - Pipelining uit*


Ik doe weer 10 tijdsmetingen zodat ik weer een goed beeld heb van de gemiddelde tijd. Hieronder kan je de tijden vinden in een tabel


| Tijdsmeting |Tijd (Plugin weergave - Seconden)               |
| :---           | :---           |
| Tijdsmeting 1  | 0:00:20,546 - 20,546 seconden     | 		
|Tijdsmeting 2 | 0:00:20,620 - 20,620 seconden         | 
| Tijdsmeting 3 |0:00:20,772 - 20,772 seconden        | 
|Tijdsmeting 4 | 0:00:20,867 - 20,867 seconden     | 
| Tijdsmeting 5| 0:00:20,787 - 20,787 seconden    | 
| Tijdsmeting 6  | 0:00:20,881 - 20,881 seconden     | 		
|Tijdsmeting 7 | 0:00:21,018 - 21,018 seconden         | 
| Tijdsmeting 8 |0:00:20,738 - 20,738 seconden        | 
|Tijdsmeting 9 | 0:00:20,727 - 20,727 seconden     | 
| Tijdsmeting 10| 0:00:21,067 - 21,067 seconden    | 

*Tabel 3: Tijdsweergave SSH pipelinning*

Hieruit kunnen we afleiden dat de gemiddelde tijd met SSH pipelining 20,802 seconden is, de mediaan is 20,779 seconden.


## ControlPersist

ControlePersist is een SSH-functie die de verbinding met de server open houdt, klaar om deze te hergebruiken tussen aanroepingen. Johnson vertelt in het artikel dat het default Control_path meer dan 102 karakters lang is. En door deze te veranderen naar de */tmp* een speed improvment zal krijgen.

Dit doe ik door in het **ansible.cfg** bestand onder de lijn van pipelining het volgende toevoeg `` control_path = /tmp/ansible-ssh-%%h-%%p-%%r ``.

Ik test weer 10 keer de tijdsafmetingen, hieronder de tabel. 

| Tijdsmeting |Tijd (Plugin weergave - Seconden)               |
| :---           | :---           |
| Tijdsmeting 1  | 0:00:20,318 - 20,318 seconden     | 		
|Tijdsmeting 2 | 0:00:19,741 - 19,741 seconden         | 
| Tijdsmeting 3 |0:00:20,914 - 20,914 seconden        | 
|Tijdsmeting 4 | 0:00:20,885 - 20,885 seconden     | 
| Tijdsmeting 5| 0:00:20,946 - 20,946 seconden    | 
| Tijdsmeting 6  | 0:00:20,876 - 20,876 seconden     | 		
|Tijdsmeting 7 | 0:00:20,924 - 20,924 seconden         | 
| Tijdsmeting 8 |0:00:20,738 - 20,738 seconden        | 
|Tijdsmeting 9 | 0:00:20,767 - 20,767 seconden     | 
| Tijdsmeting 10| 0:00:20,811 - 20,811 seconden    | 

*Tabel 4: Tijdsweergave ControlPersist*

Hieruit kunnen we afleiden dat de gemiddelde tijd met SSH pipelining en ControlPersist 20,692 seconden is, de mediaan is 20,843 seconden.

## Async

Johnson spreekt in zijn artikel ook nog over een 3de stap. Namelijk async. Met Async tasks kan je terwijl er 1 lange taak aan het runnen is, ook nog een andere klein uitvoeren. 

Bij mijn setup zou dit moeilijk werken aangezien je met post en pre tasks werkt. Aan de tijden te zien is er misschien wel een window bij het kopieren van de ansible.cfg file waar je acl's kunt aanmaken. Maar aangezien het dat na het overlopen van het playbook moet gebeuren en het kopieren ervoor, gaat dit niet. Ook is dit meer voor tasks die een lange tijd duren. Deze beide tasks duren maar een halve second.

![file](https://github.com/MaartenDeS/elnx-sme/blob/soluation/Actua/Foto's/file.png)
*Foto 5: Tijdsduur Kopie Tasks*


![acl](https://github.com/MaartenDeS/elnx-sme/blob/soluation/Actua/Foto's/acl.png)
*Foto 6: Tijdsduur ACL Tasks*


## Bronnen
Johsnon - 2015: <https://adamj.eu/tech/2015/05/18/making-ansible-a-bit-faster>

jlafon - 2016: <https://github.com/jlafon/ansible-profile>

Ansible.cfg file manage: <http://itmithran.com/installation-configuration-ansible/>

Uitleg SSH pipelining: <http://toroid.org/ansible-ssh-pipelining>

Check Pipelining Aan: <https://stackoverflow.com/questions/43438519/check-if-ansible-pipelining-is-enabled-working>

Voorbeeld Ansible.cfg file: <https://github.com/opentable/ansible-examples/blob/master/ansible.cfg>

Async: <http://acalustra.com/acelerate-your-ansible-playbooks-with-async-tasks.html>

