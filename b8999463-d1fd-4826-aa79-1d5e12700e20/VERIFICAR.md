# Cómo verificar este certificado

Esto se verifica **sin la herramienta que lo generó**, con tres programas libres:
`shasum` (o `sha256sum`), `gpg` y `ots`. Si los tres pasos dan bien, está probado que
**esta obra exacta existía, declarada por su autor, antes de la fecha del sello**.

Obra: **Nosto Estudios — Wordmark Blanco**
Certificado: `b8999463-d1fd-4826-aa79-1d5e12700e20`
Página pública: https://alexfbertini.github.io/certificados-nosto/b8999463-d1fd-4826-aa79-1d5e12700e20

## Los archivos

| archivo | qué es |
|---|---|
| `obra.svg` | la obra, byte a byte |
| `manifiesto.json` | los datos del certificado, para máquinas |
| `manifiesto.txt` | los mismos datos, legibles (copia de cortesía) |
| `manifiesto.txt.asc` | **el manifiesto firmado — es la copia que vale** |
| `manifiesto.txt.asc.ots` | el sello de tiempo del manifiesto firmado |
| `estado.json` | el estado del sello (cambia con el tiempo, no está firmado) |
| `certificado.pdf` | representación visual; se puede borrar y regenerar |

La cadena es: **obra → manifiesto firmado → sello**. El PDF no es prueba de nada;
si lo perdés no perdés nada. Si perdés `obra.svg` byte a byte, se pierde todo: el
hash ya no se puede reproducir.

## Paso 1 — que el archivo sea el que dice el manifiesto

```sh
shasum -a 256 obra.svg      # en Linux: sha256sum obra.svg
```

Tiene que dar exactamente:

```
aa90bb9617feae3a3b7112128fc9f9150b6b1a2347b93d79c8995c5daa720fd3
```

Ese mismo hash figura dentro de `manifiesto.txt.asc`, que está firmado. Si cambia
un solo byte del SVG, el hash cambia y este paso falla.

## Paso 2 — que el manifiesto lo haya firmado el autor

```sh
gpg --import clave-publica.asc      # o: gpg --recv-keys C49EAEEAEBE4DC7AA0D8A44F5808DD7D5164DF8E
gpg --verify manifiesto.txt.asc
```

Tiene que decir **Good signature** (o «Firma correcta») de la clave con huella:

```
C49EAEEAEBE4DC7AA0D8A44F5808DD7D5164DF8E
```

GnuPG va a avisar que la clave no es «de confianza»: eso es normal y no es un error. Significa que nadie de tu red firmó esa clave, no que la firma sea mala. Lo que importa es que la huella sea exactamente la de arriba.

Verificá también que el texto firmado diga lo que tiene que decir: abrilo con
cualquier editor (`manifiesto.txt.asc` es texto plano) y leelo. **Lo que vale es
el texto que está adentro del `.asc`**, no el `manifiesto.txt` suelto.

## Paso 3 — que ese manifiesto firmado ya existía en tal fecha

```sh
pipx install opentimestamps-client       # o: pip install opentimestamps-client
```

Estado al momento de generar este archivo: enviado a los calendarios de OpenTimestamps y **pendiente de confirmación**. La confirmación suele tardar entre unas horas y un día, y ocurre sola: el `.ots` se completa cuando el calendario publica su prueba en Bitcoin.

### Si tenés un nodo Bitcoin

```sh
ots verify manifiesto.txt.asc.ots
```

Responde `Success! Bitcoin block N attests existence as of ...`. Esta es la
verificación que no depende de nadie: la hace tu propio nodo.

### Si no tenés un nodo (lo normal)

`ots verify` **falla sin un nodo Bitcoin local** (dice `Could not connect to
Bitcoin node`). Eso no es un problema del sello: es que el programa no tiene con
qué leer la cadena. El mismo control se hace en dos pasos:

```sh
ots info manifiesto.txt.asc.ots | tail -3
```

Las dos últimas líneas dicen en qué bloque está y con qué raíz de Merkle:

```
verify BitcoinBlockHeaderAttestation(358391)
# Bitcoin block merkle root <64 caracteres hexadecimales>
```

Ese dato sale del archivo `.ots` y es criptográfico: la cadena de hashes que
lleva del manifiesto firmado hasta esa raíz está toda dentro del archivo. Falta
una sola cosa, que es mirar la cadena de bloques y confirmar que el bloque
tiene esa raíz:

```sh
BLOQUE=358391
HASH=$(curl -s https://blockstream.info/api/block-height/$BLOQUE)
curl -s https://blockstream.info/api/block/$HASH | python3 -m json.tool | grep -E 'merkle_root|timestamp'
```

Si el `merkle_root` es idéntico al que imprimió `ots info`, el sello es bueno:
el manifiesto firmado ya existía cuando se minó ese bloque. `timestamp` es la
fecha del bloque, en segundos desde 1970. También sirve `mempool.space` o
cualquier otro explorador: son datos públicos y todos tienen que coincidir.

Nadie —tampoco el autor— puede meter un dato en un bloque ya minado.

## Qué prueba y qué no prueba esto

**Prueba** que el archivo con ese SHA-256 existía con ese contenido exacto antes
del bloque sellado, y que quien controla esa clave declaró ser su autor.

**No prueba** derechos de marca ni sustituye un registro marcario.
Marca solicitada ante la DNPI (Uruguay), expediente 590460 en examen.

Autor: Alex Flores Bertini — Punta del Este, Uruguay
Titular: Alex Flores Bertini / Nosto Estudios
Declarado: 9 de setiembre de 2026, 06:58 UTC
