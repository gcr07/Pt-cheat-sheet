# Pt-cheat-sheet

# Revision de codigo


## retire.js

Es una extension de google chrome parecida a wapalyzer que se encarga de "Scanning website for vulnerable js libraries"


## Horusec

Permite revisar codigo 

> orusec is an open source tool that performs static code analysis to identify security flaws during the development process. Currently, the languages for analysis are: C#, Java, Kotlin, Python, Ruby, Golang, Terraform, Javascript, Typescript, Kubernetes, PHP, C, HTML, JSON, Dart, Elixir, Shell, Nginx, Swift. The tool has options to search for key leaks and security flaws in all files of your project, as well as in Git history. Horusec can be used by the developer through the CLI and by the DevSecOps team on CI /CD mats.

> https://horusec.io/site/

## Visual code grepper

isual Code Grepper is an automated code security review tool that handles C/C++, Java, C#, VB and PL/SQL. It has a few features that should hopefully make it useful to anyone conducting code security reviews, particularly where time is at a premium:
1. In addition to performing some more complex checks it also has a config file for each language that basically allows you to add any bad functions (or other text) that you want to search for.

## SonarQube

SonarQube es una plataforma para evaluar código fuente. Es software libre y usa diversas herramientas de análisis estático de código fuente como Checkstyle, PMD o FindBugs para obtener métricas que pueden ayudar a mejorar la calidad del código de un programa.​

## 

Comandos pt infra
-------------------------------
Nota
Recuerda leer y entender el nmap, no es como los pt web, aqui es un rompecabezas( por parte de la expereiencia vas entender por cuales puertos empezar).
---------------------------



Reconocimiento
NMAP



nmap -p- -sS --min-rate 5000 --open  -vvv -n -Pn



nmap -sC -sV -p



namp -p- -sU
nmap -sU -O -p- -oA udp



Nota: recordar leer bien lo que salga del nmap



SE empezara primero con los smb y la parte http, si en el escaneo encontramos un puerto de  dns, kerberos  y  ldap o AD, se esta atacando un AD y el vector de ataque es dirigido a AD



Ataque para AD
https://wadcoms.github.io/




------------------------------------------------------------------------------------------------------------------------------



SMB Puertos 445,139,135
Nota se sabe que tambien el 135 o lo que se tenga que ver con  msrpc es smb, devido a esto se agrega en la enumeracion de smb.
    Dentro del escaneo con nmap se puedo encontrar las versiones de smb pero buscamos vas como usurios o carpetas compartidas



****Herremientas



enum4linux -a IP
smbclient --no-pass -L //<IP>
smbclient -L \\\\ip\\
nmap --script "safe or smb-enum-*" -p 445 <IP>
sudo smbmap -R Folder -H <IP> -A <FileName>
nmap --script smb-brute -p 445 <IP>
smbmap -H activo.htb
use auxiliary/scanner/smb/smb_lookupsid
use auxiliary/scanner/smb/pipe_auditor




https://tpx.mx/blog/2020/smb-attacks-for-dummies.html
https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb



------------------------------------------------------------------------------------------------------------------------------
DNS (dominio) 53
Nota: Regularmente dentro del escaneo de nmap se encuentra la parte de donimio o subdominio,por otro lado se tendra que validar para encontrar mas informacion.



*******Herramientas



nslookup ip
gobuster dns -d domain.local -t 25 -w /opt/Seclist/Discovery/DNS/subdomain-top2000.txt
nmap -n --script "(default and *dns*) or fcrdns or dns-srv-enum or dns-random-txid or dns-random-srcport" <IP>
auxiliary/gather/enum_dns
nmap -sSU -p53 --script dns-nsec-enum --script-args dns-nsec-enum.domains=paypal.com ns3.isc-sns.info
dnsrecon -D subdomains-1000.txt -d <DOMAIN> -n <IP_DNS>
dnscan -d <domain> -r -w subdomains-1000.txt
dnsmap nombre de dominio



https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns?q=ker
--------------------------------------------------------------------------------------------------------------------------
Kerbero  88



Nota:Si se encuentra dentro del escaneo un kermeberos, dns, smb y tambien AD o LDAp es 100% que el vector de ataque va a ser dirigido  un AD
Nota 2:A este punto ya deberiamos obtener una informacion relevante como nombre o contraeñas o carpetas compartidas.



Herramientas
nmap -p 88 --script=krb5-enum-users --script-args="krb5-enum-users.realm='DOMAIN'" <IP>
Nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm='<domain>',userdb=/root/Desktop/usernames.txt <IP>
msf> use auxiliary/gather/kerberos_enumusers
kerbrute



https://book.hacktricks.xyz/network-services-pentesting/pentesting-kerberos-88
https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/kerberos-authentication
https://book.hacktricks.xyz/windows-hardening/active-directory-methodology



https://github.com/ropnop/kerbrute/releases
------------------------------------------------------------------------------------------------------------------------------
kpasswd5? 464



Nota: ejecutando kpasswd5. Este puerto se usa para cambiar/establecer contraseñas contra Active Director
Nota2:entramos ya a un mundo que no conocemos.(Prueba y error )



nmap -p 88 --script krb5-enum-users --script-args krb5-enum-users.realm='test'



https://nmap.org/nsedoc/scripts/krb5-enum-users.html
https://ranakhalil101.medium.com/hack-the-box-active-writeup-w-o-metasploit-79b907fd4356#:~:text=Port%20464%3A%20running%20kpasswd5.,based%20network%20access%20control%20program
-----------------------------------------------------------------------------------------------------------------------------
tcpwrapped



Nota :un programa de control de acceso a la red basado en host en Unix y Linux. Cuando Nmap etiqueta algo tcpwrapped, significa que el comportamiento del puerto es consistente con uno que está protegido por tcpwrapper. Específicamente, significa que se completó un protocolo de enlace TCP completo, pero el host remoto cerró la conexión sin recibir ningún dato.




Esta protegiendo un programa



Es importante tener en cuenta que tcpwrapper protege los programas , no los puertos. Esto significa que una respuesta válida (no un falso positivo) tcpwrappedindica que hay un servicio de red real disponible, pero usted no está en la lista de hosts autorizados para hablar con él. Cuando se muestra una cantidad tan grande de puertos como tcpwrapped, es poco probable que representen servicios reales, por lo que el comportamiento probablemente signifique otra cosa.



Lo que probablemente esté viendo es un dispositivo de seguridad de red como un firewall o IPS. Muchos de estos están configurados para responder a escaneos de puertos TCP, incluso para direcciones IP que no tienen asignadas. Este comportamiento puede ralentizar un escaneo de puertos y nublar los resultados con falsos positivos.



----------------------------------------------------------------------------------------------------------------------------
LDAP 3269



Herramienta
nmap -n -sV --script "ldap* and not brute" -p 389 <DC IP>
ldapsearch(buscra la forma de  instalarlo)




https://book.hacktricks.xyz/network-services-pentesting/pentesting-ldap
https://github.com/CroweCybersecurity/ad-ldap-enum
-----------------------------------------------------------------------------------------------------------------------------
ms-wbt-server  3389
    
    
# VTP
    
 VTP son las siglas de VLAN Trunking Protocol, un protocolo de mensajes de nivel 2 usado para configurar y administrar VLANs en equipos Cisco
    
    Por ejemplo 
    
 Internet u otra red->VLAN------------Trunk----------VLAN2->Internet u otra red   
    
# Enlace Troncal 
    
 Es un enlace que se configura en uno o más puertos de un switch para permitir el paso del tráfico de las distintas VLANs que hemos configurado.
    
# Spaning Tree 
    
 En comunicaciones, STP es un protocolo de red de capa 2 del modelo OSI. Su función es la de gestionar la presencia de bucles en topologías de red debido a la existencia de enlaces redundantes.
    
    
