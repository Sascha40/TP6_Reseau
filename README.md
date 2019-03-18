# TP6_Reseau 
# Une topologie qui ressemble un peu à quelque chose, enfin ?  
A french network exercise for my computer science class.

## 1. Présentation du lab et contexte

### Réseaux IP et aires OSPF
Réseaux | `area 0` | `area 1` | `area 2` | Commentaire
--- | --- | --- | --- | ---
`10.6.100.0/30` | X | - | - | Liaison entre `r1` et `r2`
`10.6.100.4/30` | X | - | - | Liaison entre `r1` et `r4`
`10.6.100.8/30` | X | - | - | Liaison entre `r2` et `r3` 
`10.6.100.12/30` | X | - | - | Liaison entre `r3` et `r4`
`10.6.101.0/30` | - | X | - | Liaison entre `r3` et `r5`
`10.6.201.0/24` | - | X | - | Réseau des clients
`10.6.202.0/24` | - | - | X | Réseau des serveurs  


### Adressage IP de chacune des machines
Machines | `10.6.100.0/30` | `10.6.100.4/30` | `10.6.100.8/30` | `10.6.100.12/30` | `10.6.101.0/30` | `10.6.201.0/24` | `10.6.202.0/24`
--- | --- | --- | --- | --- | --- | --- | --- 
`r1.tp6.b1` | `10.6.100.1` | `10.6.100.5` | - | - | - | - | `10.6.202.254`
`r2.tp6.b1` | `10.6.100.2` | - |  `10.6.100.9` | - | - | - | -
`r3.tp6.b1` | - | - | `10.6.100.10` | `10.6.100.14` | `10.6.101.1` | - | -
`r4.tp6.b1` | - |  `10.6.100.6` | - | `10.6.100.13` | - | - | -
`r5.tp6.b1` | - | - | - | - |  `10.6.101.2` |  `10.6.201.254` | -
`client1.tp6.b1` | - | - | - | - | - |  `10.6.201.10` | -
`client2.tp6.b1` | - | - | - | - | - |  `10.6.201.11` | -
`server1.tp6.b1` | - | - | - | - | - | - | `10.6.202.10`  

## 2. Mise en place du lab  

Schema GNS  
![schema](gn3_tp6/schema.PNG)  

Mes vérifications : 


* `server1` :  
    ```
    [root@server1 ~]# sudo traceroute -I client1
    traceroute to client1 (10.6.201.10), 30 hops max, 60 byte packets
    1  gateway (10.6.202.254)  7.223 ms  7.145 ms  7.182 ms
    2  10.6.100.6 (10.6.100.6)  28.683 ms  28.351 ms  27.723 ms
    3  10.6.100.14 (10.6.100.14)  38.346 ms  37.977 ms  38.435 ms
    4  10.6.101.2 (10.6.101.2)  65.335 ms  64.884 ms  70.021 ms
    5  client1 (10.6.201.10)  70.946 ms  70.338 ms  78.237 ms
    [root@server1 ~]# sudo traceroute -I client2
    traceroute to client2 (10.6.201.11), 30 hops max, 60 byte packets
    1  gateway (10.6.202.254)  4.121 ms  4.143 ms  4.009 ms
    2  10.6.100.2 (10.6.100.2)  23.925 ms  24.073 ms  24.011 ms
    3  10.6.100.10 (10.6.100.10)  47.782 ms  47.664 ms  47.131 ms
    4  10.6.101.2 (10.6.101.2)  56.509 ms  56.849 ms  56.687 ms
    5  client2 (10.6.201.11)  63.130 ms  63.333 ms  63.855 ms
    ```  
* `client1`:  
    ```
    [root@client1 ~]# sudo traceroute -I server1
    traceroute to server1 (10.6.202.10), 30 hops max, 60 byte packets
    1  gateway (10.6.201.254)  8.254 ms  9.006 ms  8.922 ms
    2  10.6.101.1 (10.6.101.1)  18.565 ms  18.738 ms  18.455 ms
    3  10.6.100.13 (10.6.100.13)  34.649 ms  34.340 ms  35.442 ms
    4  10.6.100.5 (10.6.100.5)  49.044 ms  49.675 ms  49.915 ms
    5  server1 (10.6.202.10)  61.311 ms  61.413 ms  60.661 ms
    [root@client1 ~]# sudo traceroute -I client2
    traceroute to client2 (10.6.201.11), 30 hops max, 60 byte packets
    1  client2 (10.6.201.11)  3.042 ms  2.821 ms  2.886 ms
    ```  
* `client2`:  
    ```
    [root@client2 ~]# sudo traceroute -I server1
    traceroute to server1 (10.6.202.10), 30 hops max, 60 byte packets
    1  gateway (10.6.201.254)  6.346 ms  6.458 ms  6.144 ms
    2  10.6.101.1 (10.6.101.1)  16.568 ms  16.565 ms  16.203 ms
    3  10.6.100.13 (10.6.100.13)  37.126 ms  37.190 ms  37.112 ms
    4  10.6.100.5 (10.6.100.5)  58.820 ms  58.294 ms  58.727 ms
    5  server1 (10.6.202.10)  68.376 ms  68.652 ms  68.735 ms
    [root@client2 ~]# sudo traceroute -I client1
    traceroute to client1 (10.6.201.10), 30 hops max, 60 byte packets
    1  client1 (10.6.201.10)  1.555 ms  1.342 ms  1.591 ms
    ```  

### Lab 3 : Let's end this properly  


Service | Qui porte le service ? | Pour qui ? | Pourquoi ? 
--- | --- | --- | ---
**NAT** | `r4.tp6.b1` | tout le monde (routeurs & VMs) | Le NAT permet d'accéder à l'extérieur, il permet de sortir du LAN. Toutes les machines peuvent en avoir besoin dans notre petite infra
**Serveur Web** | `server1.tp6.b1` | réseau client `10.6.201.0/24` | Le serveur Web symbolise un service d'infra en interne. Dispo pour nos clients. Plus de détails dans la section dédiée.
**DHCP** | `client2.tp6.b1` | réseau client `10.6.201.0/24` | Le DHCP (qui permet d'attribuer des IPs automatiquement) c'est pour des clients. Pas pour des serveurs. Un serveur, on veut qu'il ait une IP fixe. 
**DNS** | `server1.tp6.b1` | tout le monde (routeurs & VMs) | Le DNS nous permettra de résoudre les noms de domaines en local et nous passer du fichier `/etc/hosts`
**NTP** | `server1.tp6.b1` | réseau serveur `10.6.202.0/24` | Le NTP, qui permet la synchronisation de l'heure, est souvent indispensable pourdes serveurs mais totalement négligeable pour des clients (genre vos PCs, s'ils sont pas à l'heure, tout le monde s'en fout)  

### 1. NAT : accès internet  
YES DU GNS ON AIME

* Ma connection internet fonctionne sur les vms :  
    ```
    [root@server1 ~]# dig google.com @8.8.8.8

    ; <<>> DiG 9.9.4-RedHat-9.9.4-72.el7 <<>> google.com @8.8.8.8
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31205
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 512
    ;; QUESTION SECTION:
    ;google.com.                    IN      A

    ;; ANSWER SECTION:
    google.com.             299     IN      A       216.58.213.174

    ;; Query time: 67 msec
    ;; SERVER: 8.8.8.8#53(8.8.8.8)
    ;; WHEN: Wed 13 16:32:19 CET 2019
    ;; MSG SIZE  rcvd: 55
    ```  

* Ma connection internet fonctionne Router je récupère de l'HTML :  
    ```
    R4#telnet trip-hop.net 80
    Trying trip-hop.net (213.186.33.4, 80)... Open
    GET /
    HTTP/1.1 
    Set-Cookie: 240planBAK=R2339303237; path=/; expires=Wed, 13-Wed-2019 17:04:14 GMT
    Server: nginx
    Date: Wed, Wed 13 17:04:21 GMT
    Content-Type: text/html; charset=utf8
    Pragma: no-cache
    Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
    Content-Length: 5329
    X-IPLB-Instance: 17296
    X-Cache: MISS from PF1-BOR1FR
    X-Cache-Lookup: MISS from PF1-BOR1FR:3128
    Connection: close

    <html xml:lang="en-GB" lang="en-GB">
        <head>
            <meta name="viewport" content="width=device-width">
            <title qtlid="283606">Site not installed</title>
            <meta http-equiv="refresh" content="480">
            <meta http-equiv="Content-Type" content="text/html; charset=utf-8">  
            ...
    ```  

### 2. Un service d'infra


* Depuis `server1` :  
    ```
    [root@server1 etc]# curl localhost
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

    <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
        <head>
            <title>Test Page for the Nginx HTTP Server on Fedora</title>
            <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
            <style type="text/css">
                /*<![CDATA[*/
                body {
                    background-color: #fff;
                    color: #000;
                    font-size: 0.9em;
                    font-family: sans-serif,helvetica;
                    margin: 0;
                    padding: 0;
                }
                :link {
                    color: #c00;
                }
                :visited {
                    color: #c00;
                }
                a:hover {
                    color: #f50;
                }
                h1 {
                    text-align: center;
                    margin: 0;
                    padding: 0.6em 2em 0.4em;
                    background-color: #294172;
                    color: #fff;
                    font-weight: normal;
                    font-size: 1.75em;
                    border-bottom: 2px solid #000;
                }  
                ...
    ```  
    
    
* Depuis `client1` :  
    ```
    [root@client1 ~]# curl server1
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

    <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
        <head>
            <title>Test Page for the Nginx HTTP Server on Fedora</title>
            <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
            <style type="text/css">
                /*<![CDATA[*/
                body {
                    background-color: #fff;
                    color: #000;
                    font-size: 0.9em;
                    font-family: sans-serif,helvetica;
                    margin: 0;
                    padding: 0;
                }
                :link {
                    color: #c00;
                }  
                ...
    ```  


### 3. Serveur DHCP
+ Modification de l'ip de `client1` en ip dynamique:

    ```
    [root@client1 ~]# sudo dhclient -v -r
    Internet Systems Consortium DHCP Client 4.2.5
    Copyright 2004-2013 Internet Systems Consortium.
    All rights reserved.
    For info, please visit https://www.isc.org/software/dhcp/

    Listening on LPF/enp0s8/08:00:27:1c:7e:6d
    Sending on   LPF/enp0s8/08:00:27:1c:7e:6d
    Listening on LPF/enp0s3/08:00:27:ab:c9:98
    Sending on   LPF/enp0s3/08:00:27:ab:c9:98
    Sending on   Socket/fallback
    ```
    `ip a` : 
    ``` 
        enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 08:00:27:ab:c9:98 brd ff:ff:ff:ff:ff:ff
        inet 10.6.201.50/24 brd 10.6.201.255 scope global noprefixroute dynamic enp0s3
    ```
    
 `Apparement ça a marché`  

### 4. Serveur DNS  


Sur `server1`:  
    * Pour installer le serveur DNS : `sudo yum install -y bind*`  
    * J'ai édité le fichier de configuration : `sudo vi /etc/named.conf`  
    * Puis les fichiers de zone : `sudo vi /var/named/forward.tp6.b1` puis `sudo nano /var/named/reverse.tp6.b1` 
      J'ai bien changé les IPs à cause du DHCP  
    * Nouveau propriétaire : `sudo chown named:named /var/named/*tp6.b1`  
    * Ouverture des ports firewall : `sudo firewall-cmd --add-port=53/tcp --permanent` ainsi que `sudo firewall-cmd --add-port=80/udp --permanent`
 
### 5. Serveur NTP 

* vérifier que chrony est présent

    
    
