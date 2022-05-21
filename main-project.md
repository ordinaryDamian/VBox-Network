

<style>


h1 { counter-reset: h2counter; }
h2 { counter-reset: h3counter; }
h3 { counter-reset: h4counter; }
h4 { counter-reset: h5counter; }
h5 { counter-reset: h6counter; }
h6 {}

h2:before {
    counter-increment: h2counter;
    content: counter(h2counter) ".\0000a0\0000a0";
}

h3:before {
    counter-increment: h3counter;
    content: counter(h2counter) "." counter(h3counter) ".\0000a0\0000a0";
}

h4:before {
    counter-increment: h4counter;
    content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) ".\0000a0\0000a0";
}

h5:before {
    counter-increment: h5counter;
    content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) "." counter(h5counter) ".\0000a0\0000a0";
}

h6:before {
    counter-increment: h6counter;
    content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) "." counter(h5counter) "." counter(h6counter) ".\0000a0\0000a0";
}
img[src*='#left'] {
    float: left;
}
img[src*='#right'] {
    float: right;
}
img[src*='#center'] {
    display: block;
    margin: auto;
}
</style>

# Nastavenia sietových kariet vo Virtual Box

<center>Tento referát sa zaoberá sieťovými nastaveniami v programe Virtual Box</center>
<center>Vypracoval: Damián Nadžady 3.A 13.5.2022</center>

## Čo je Virtual Box a čo su jeho nastavenia

Virtual Box (type-2 hypervisor) je program v ktorom používateľ môže simulovať virtulany počítač ktorý bezi v takzvanom sandboxe a je oddelený od hlavného počitača ktorý beží priamo na komponentoch. Virtual Box beží ako program to znamená že o požiadavky na prístup k komponentom (CPU, RAM, ...) prechádzaju cez OS ktorý beží na komponentoch.



### VB - Virtual box ma mnoho nastavený

![echo PATH](./obrazky/VirtualBox_JDvy6DNgF6.png)

Jedno z podstatných nastavený je nastavenie sietovej karty v danom virtualnom PC
### Sieťové nastavenia vo Virtual Boxe
![echo PATH](./obrazky/VirtualBox_CG9kXvP51p.png)

V záložke Network máme niekoľko adaptérov ktoré si vieme zapnút, to môžeme chápať ako 4 rozdielne sietové kartky na ktoré si vieme nastaviť iné módy ktoré sú priradené k fyzickým komponentom.

V ďalšej ponuke si vieme vybrať cez ktorý fyzicky interface chceme tento PC nechať komunikovať.

V ponuke Advanced si môžeme nastaviť port-forwarding alebo virtualnu MAC adresu daného adaptéra, plus niekoľko dalších nastavený.  

### Typ adaptéra

Po rozklinutý položky Advanced si môžeme vybrat aj adaptér ktorý sa bude zobrazovať v samotnom guest OS

<ul style="list-style-type:circle;">
  <li>AMD PCNet PCI II (Am79C970A)</li>
  <li>AMD PCNet FAST III (Am79C973)</li>
  <li>Intel PRO/1000 MT Desktop (82540EM), <b>Default</b></li>
  <li>Intel PRO/1000 T Server (82543GC)</li>
  <li>Intel PRO/1000 MT Server (82545EM)</li>
  <li>Paravirtualized network adapter (virtio-net)</li>
</ul>


#### NAT <b>Default</b>
NAT - Network address translation

Najjednoduchší sposob ako sa cez guest PC pripojiť na internet a surfovať externé stránky. Nevyžaduje si žiadne nastavenia a funguje takzvane out of the box. Je zvolený ako prevolený režim virtualnej sietovej karty.

Guest OS sa chová ako keby to bol PC za routerom na privatnej siety, tento router je v realite __Oracle VM VirtualBox NAT Engine__ ktorý dostane packety z Guesta a znova ich zabali a posle na router z OS vo fyzickom PC.

Nevýhodou je že takýto Guest OS nie je vidieť separatne na internete a ak sa k nemu chceme pripojit musíme nastavit portforwarding.

Guest OS dostane DHCP server a iné podstatné adresy z _Oracle VM VirtualBox NAT Engine_ ktorý sa chová ako router na klasickej siety. IP adresa sa prideluje z inej siete ako ta na ktorej je fyzický PC. Adresa začína 10.0.2.0 pre prvý interface, 10.0.3.0 predruhy int a tak dalej.

![Markdown Logo](https://www.nakivo.com/blog/wp-content/uploads/2019/07/VirtualBox-network-settings-%E2%80%93-the-NAT-Network-mode.webp)
___

#### Bridged adapter

Používaťel si vie zvoliť cez ktorý fyzický adaptér sa bude trafica na siety posielať

Oracle VM VirtualBox využíva _net filter driver_ na to aby oddelil dáta ktoré sú smerované pre Guest OS. To znamená že na siety sa vytvorý virtualne rozhranie ktoré funguje ako další PC na siety, Guest OS dostane od DHCP všetko potrebné pre komunikáciu s routerom. Guest OS vkladá data fyzickému rozhraniu ktoré ho zabalí za svoje. To znamená že komunikácia na súkromnej siety medzi virtualnym OS a ostatnými PC s IP adresamy je možné.

![Markdown Logo](https://www.nakivo.com/blog/wp-content/uploads/2019/07/VirtualBox-network-settings-%E2%80%93-bridged-networking.webp)
___

#### Internal Network

Funguje podobne ako Bridge adapter s rozdielom tým že tento privatna sieť slúži iba pre Guest OS na jednom fyzickom PC.
Táto sieť je použitá ak chcem mať viac virtuálnych PC na jednom fyzickom PC a chceme aby spolu komunikovali ale aby sa táto komunikacia nedostala k iným hostom na privatnej sieti fyzického PC.
Pri Bridge adaptéry idú všetky packety cez fyzické rozhranie rozhranie host OS a preto sa dajú zaznamenať napr. cez Wireshark, takto tomu vie me zabrániť. 
Pei vytváraný si vyberáme meno vďaka ktorému sa sieť identifikuje.
Po čo __Oracle VM VirtualBox support driver__ detekuje další Guest OS s rovnakým network ID tak ich automatický pripojí a začne sa chovať ako _Switch_.
Pre pripojenie na sieť možeme buď zvolit vstavaný DHCP server ale priradiť statické IP adresy.

![Markdown Logo](https://www.nakivo.com/blog/wp-content/uploads/2019/07/VirtualBox-network-settings-%E2%80%93-using-the-Internal-network-mode-in-a-combination-with-the-NAT-mode.webp)
___

#### Host-only adapter

Funguje ako zmiešanie Bridge a internal adaptérov.
__Oracle VM VirtualBox__ vytvorý nový softwareový loopback interface hneď veďla fyzického interfacu.
Pre jeden Guest OS možeme nastaviť Internal network aby vedel komunikovať iba vrámci host OS s iným Guest OS.
Na druhom Guest OS nastavýme Host-Only, to znamená že druhý Guest OS vie komunikovať aj na internal network s prvým Guest OS ale zároveň ma aj IP adresu ako Bridge na privatnej sieti host OS.

![Markdown Logo](https://www.nakivo.com/blog/wp-content/uploads/2019/07/VirtualBox-network-settings-%E2%80%93-VMs-use-the-host-only-network.webp)
___

#### Generic driver



#### NAT Network



#### Cloud Network

Najnovšie nastavenie, je v štádiu vývoja Oracle ho označuje za __Experimentálne__. 

#### Bez adaptéra

Toto nastavenie znamená že Guest OS nedokaže komunikovať vôbec, jeho sieťová karta neexistuje/nie je pripojená a preto jeho pristup na akukoľvek sieť a jeho komunikacia je úplne znemožnená.
___

<!-- Obsah vygenerovať neskor
ak mas extension tak CTRL+SHIFT+P a napisat create table of content
-->