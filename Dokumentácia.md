# Dokumentácia Projektu

## Úvod

Tento projekt demonštruje konfiguráciu VLAN a DHCP na sieti s routerom Cisco (2911) a prepínačmi (2960-24TT) spolu s IP telefóniou. Sieť obsahuje 10 kancelárií, každá s jedným PC a IP telefónom. Cieľom je nakonfigurovať VLAN pre hlasovú a dátovú komunikáciu, zabezpečiť priradenie IP adries pomocou DHCP a funkčnú VoIP linku.

## Topológia Sieťe

- **Router (Cisco 2811):**
	- Rozdelené subrozhrania pre VLAN 10 (Voice) a VLAN 20 (Data).
- **Hlavný prepínač (2960-24TT):**
	- Pripojený k routeru cez trunk.
	- Pripojený k 10 podřazeným prepínačom.
- **Podradené prepínače (2960-24TT):**
	- Pripojený k routeru a prepínaču cez trunk.
	- Každý pripojený k PC a IP telefónu.

![image](https://s.0ch.fun/hal_pcv_cptprojekt_iptelephony/img1.png)

Obrázok 1: Snímka logickej typológie siete

## **Konfigurácia siete**


### Router (2811)

1. **Konfigurácia subrozhraní pre VLAN:**

    ```typescript
    interface fastEthernet0/0.10
    encapsulation dot1Q 10
    ip address 192.168.10.1 255.255.255.0
    
    interface fastEthernet0/0.20
    encapsulation dot1Q 20
    ip address 192.168.20.1 255.255.255.0
    ```

2. **Nastavenie DHCP Pools**
	- Voice VLAN (VLAN 10)

    ```typescript
    ip dhcp pool Voice
    network 192.168.10.0 255.255.255.0
    default-router 192.168.10.1
    option 150 ip 192.168.10.1
    ```

	- Data VLAN (VLAN 20)

    ```typescript
    ip dhcp pool Data
    network 192.168.20.0 255.255.255.0
    default-router 192.168.20.1
    ```

3. **Vylúčenie adries:**

    ```other
    ip dhcp excluded-address 192.168.10.1 192.168.10.2
    ip dhcp excluded-address 192.168.20.1 192.168.20.2
    ```


### Hlavný prepínač (2960-24TT)

1. **Vytvorenie VLAN:**

    ```other
    vlan 10
    name Voice
    
    vlan 20
    name Data
    ```

2. **Nastavenie trunk portu:**

    ```other
    interface gigabitEthernet0/1
    switchport mode trunk
    switchport trunk allowed vlan 10,20
    ```


	Trunk port nám dovoluje 


3. **Konfigurácia prístupových portov:**

    ```other
    interface fastEthernet0/1
    switchport mode access
    switchport access vlan 20
    switchport voice vlan 10
    ```


	Pre konfiguráciu každého z podradených prepínačov budeme potrebovať ich zapojenie do hlavného prepínača. Tie su pripojené do `fastEthernet0/1` az `fastEthernet0/10`. 

### Podradené prepínače (2960-24TT)

1. **Pripojenie k hlavnému prepínaču:**

    ```other
    interface gigabitEthernet0/3
    switchport mode trunk
    switchport trunk allowed vlan 10,20
    ```

2. **Konfigurácia prístupových portov pre PC a IP telefóny:**

	V sieti sú počítače pripojené ku podradenému prepínaču do portu `fastEthernet0/1`. IP Telefóny sú pripojené do portu `fastEthernet0/2`. Pripojenie ku celo-kancelárnej siete (t.j. Hlavnému prepínaču) zabezpečuje port `fastEthernet0/3`. 


	- Konfigurácia pre PC

    ```other
    interface fastEthernet0/1
    switchport mode access
    switchport access vlan 20
    ```

	- Konfigurácia pre IP Telefón

    ```other
    interface fastEthernet0/2
    switchport mode access
    switchport access vlan 10
    ```


## Konfigurácia VoIP

1. **Nastavenie Cisco Unified Communications Manager Express** 
	- Povolenie `telephony-service` 

    ```typescript
    telephony-service
    max-dn 10
    max-ephones 10
    ip source-address 192.168.10.1 port 2000
    auto assign 1 to 10
    create cnf-files
    exit
    ```


	V týchto príkazoch speficikujeme množstvo telefónov a čísel predvolieb v sieti, v našom prípade 10. Ďalšim príkazom,  `ip source-address`, nastavujeme IP adresu routra a port kde prebieha SCCP komunikácia s VoIP zariadeniami. Príkazom auto assign automaticky priradíme "e-phone" čísla ku "dn" predvolbám. A na koniec príkaz `create cnf-files` vytvorí konfiguračné súbory pre telefóny.


	- Nastavenie DN

    ```typescript
    ephone-dn 1
    number 1001
    exit
    ```


	S každym telefónom na sieti by sme mali asociovať Phone Extension. Týmto číslom môžu ostatný účastníci našej telefonnej siete volať na specifický telefón. Tieto príkazy musíme opakovať pre všetky telefóny.


	- Nastavenie `e-phone` 

    ```typescript
    ephone 1
    mac-address 0011.2233.4455
    type 7960
    button 1:1
    exit
    ```


	V tomto kroku nastavujeme MAC adresu, typ a "button" predvolbu telefónom na sieti. Pre tento krok budeme potrebovať MAC adresy telefónov, ktoré môžme nájsť v pri podržaní myši alebo pomocou príkazu `show mac address`. Tento krok je potrebné opakovať s každým telefónom.


2. **Pripojenie IP telefónu**
	- Zariadenie musí byť pripojené k príslusnému portu k prepínaču (port na prepínači`fastEthernet0/2`)
	- V konfiguračnom okne zariadenia musí byť zariadenie pripojené ku zdroju elektrickej energie.

![image](https://s.0ch.fun/hal_pcv_cptprojekt_iptelephony/img2.png)


	2. **Automatická konfigurácia cez DHCP** 
		- IP telefóny by mali automaticky získať IP adresu z DHCP Pool pre VLAN 10.
		- Poznámka: Tento výsledok sa nedal v tejto práci dosiahnuť.
		- Na obrazovke zariadenia overíme či zobrazuje priradenú IP adresu a gateway.


----

## Problémy s konfigurovanim siete


Ako stanovuje bod č. 2.2. konfigurácie VoIP, v tejto práci sa autorovi nepodarilo automaticky získať IP adresu pre tieto zariadenia. Po hodinách strávenych "scrollovaní" stránok akými su StackOverflow alebo StackExchange autor našiel možné riešenia. Problém môže spočívať v konfigurácii TFTP servera alebo VLAN pripojení.

### Pokus 1: Manuálne nastavenie VoIP zariadenia


Podľa fóra [Cisco Community](https://community.cisco.com/t5/unified-communications-infrastructure/ip-phone-stuck-in-configuring-ip/td-p/779939) sa dá telefón nastaviť aj manuálne pomocou jeho resetovania alebo otvorenia nastavení. To docielime pomocou držania tlačidla `settings`. Toto riešenie bolo neúšpešné a autorovi ani tejto práci nedodalo smerudajné ovocie.

### Pokus 2: Zmena VLAN pripojení


Autor sa pokúsil zmeniť VLAN pripojenie telefónov. Autor sa pokúšal pripojiť telefóny pomocou VLANu 20 v móde access, VLANu 10 v móde voice. Ale aj v konfigurácií jediného módu voice alebo access. Ani tieto pokusy nedali projektu úšpešný koniec.

### Pokus 3: začať od znova


Autor sa pokúsil začať čistý projekt odznova len s VoIP zariadeniami a aj v konfigurácií s Počítačmi. Tieto snahy naukázali svetlo na konci tunela. Za zmienku stojí tento tutoriál webu [packettracernetwork.com](https://www.packettracernetwork.com/tutorials/voipconfiguration.html) ale aj iné od toho istého autora.

### Alternatívne teoretické riešenia

- Testovanie v inej verzií Programu Packet Tracer
- Použitie separátneho TFTP servera mimo routera.

## Specifické dôvody konfigurácie


### Konfigurovanie TFTP servera na Routeri


Konfigurovanie TFTP servera na routeri znižuje komplexitu pre malé siete ako je táto.

### Použitie VLAN 10 a 20


Network Traffic pre hlasové služby a dáta musia byť izolované z bezpečnostných dôvodov. A však útočnik by sa vedel v tejto konfigurácií dostať do sieťe pre Telefóny cez zapojenie do portu `fastEthernet0/2` na kancelárskom prepínači.
