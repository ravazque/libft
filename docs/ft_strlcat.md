# ft_strlcat - Guía de Funcionamiento

## Prototipo

```c
size_t ft_strlcat(char *dst, const char *src, size_t dstsize)
```

## Qué hace la función

Concatena src al final de dst sin sobrepasar el tamaño dstsize, asegurando que dst siempre termine en '\0'.

El valor de retorno es SIEMPRE `strlen(dst_inicial) + strlen(src)`, independientemente de cuánto haya copiado realmente.

## La comprobación inicial

```c
if (lngdst >= dstsize)
    return (dstsize + lngsrc);
```

### Por qué existe esta comprobación

Esta comprobación detecta dos situaciones problemáticas:

**Situación 1: Buffer mal dimensionado**
```c
char dst[10] = "Hola mundo completo";  // 19 caracteres en un buffer de 10
ft_strlcat(dst, "extra", 10);
```
Aquí `lngdst = 19` pero `dstsize = 10`. Esto es imposible: el string no puede ser más largo que su propio buffer. La función detecta que algo está mal.

**Situación 2: Buffer completamente lleno**
```c
char dst[10] = "123456789";  // 9 caracteres + '\0' = 10 bytes (buffer lleno)
ft_strlcat(dst, "extra", 10);
```
Aquí `lngdst = 9` y `dstsize = 10`. No hay espacio para añadir nada porque el único byte libre es el que ocupa el '\0' final.

### Por qué retorna dstsize + lngsrc

Cuando `lngdst >= dstsize`, la función no puede saber cuál es la longitud real de dst (puede estar corrupto o mal medido). Por eso asume que dst tiene longitud `dstsize` y retorna como si intentara concatenar a partir de ahí:

```
Retorno = dstsize + lngsrc
```

Esto permite al llamador detectar que algo falló comparando el retorno con dstsize.

## Cómo concatena

```c
while (src[i] != '\0' && (lngdst + i) < (dstsize - 1))
{
    dst[lngdst + i] = src[i];
    i++;
}
dst[lngdst + i] = '\0';
```

### La condición del while explicada

El while tiene dos condiciones que deben cumplirse simultáneamente:

**Condición 1: `src[i] != '\0'`**

No hemos llegado al final de src. Simple.

**Condición 2: `(lngdst + i) < (dstsize - 1)`**

Aquí está la clave del funcionamiento seguro. Vamos a desglosarlo:

- `lngdst` es la posición donde empieza a escribir (el final actual de dst)
- `i` es cuántos caracteres de src ha copiado hasta ahora
- `lngdst + i` es la posición donde va a escribir el siguiente carácter
- `dstsize - 1` es la última posición donde puede escribir un carácter normal

### Por qué dstsize - 1

El buffer tiene `dstsize` posiciones (índices 0 a dstsize-1). La última posición DEBE ser '\0'. Por tanto:

```
Posiciones disponibles para caracteres normales: 0 a (dstsize - 2)
Posición reservada para '\0': (dstsize - 1)
```

Ejemplo con dstsize = 10:

```
Índices:    [0][1][2][3][4][5][6][7][8][9]
Contenido:  [ ][ ][ ][ ][ ][ ][ ][ ][ ][\0]
                                        ^
                                        Siempre '\0'
```

Solo puede escribir caracteres normales en las posiciones 0 a 8. La posición 9 es obligatoriamente '\0'.

Por eso la condición es `< (dstsize - 1)`:
- Si escribo en posición 8, puedo continuar (8 < 9)
- Si intento escribir en posición 9, se detiene (9 NO es < 9)

### Ejemplo paso a paso

```c
char dst[10] = "Hola";  // lngdst = 4, dstsize = 10
char src[] = " mundo!"; // lngsrc = 7
```

Estado inicial:
```
dst: [H][o][l][a][\0][ ][ ][ ][ ][ ]
      0  1  2  3  4   5  6  7  8  9
```

Proceso de copia:

```
i=0: lngdst + i = 4 + 0 = 4
     4 < 9 (dstsize-1) → TRUE
     Copia src[0]=' ' en dst[4]
     dst: [H][o][l][a][ ][\0][ ][ ][ ][ ]

i=1: lngdst + i = 4 + 1 = 5
     5 < 9 → TRUE
     Copia src[1]='m' en dst[5]
     dst: [H][o][l][a][ ][m][\0][ ][ ][ ]

i=2: 4 + 2 = 6 < 9 → TRUE, copia 'u'
i=3: 4 + 3 = 7 < 9 → TRUE, copia 'n'
i=4: 4 + 4 = 8 < 9 → TRUE, copia 'd'

i=5: lngdst + i = 4 + 5 = 9
     9 < 9 → FALSE
     STOP: no copia 'o' ni '!'
```

Estado final antes de añadir '\0':
```
dst: [H][o][l][a][ ][m][u][n][d][ ]
      0  1  2  3  4  5  6  7  8  9
                                i=5 (no copió)
```

Luego añade '\0':
```c
dst[lngdst + i] = '\0';  // dst[4 + 5] = dst[9] = '\0'
```

Resultado final:
```
dst: [H][o][l][a][ ][m][u][n][d][\0]
      0  1  2  3  4  5  6  7  8  9
```

## Por qué el retorno es diferente de lo copiado

```c
return (lngdst + lngsrc);  // Retorna 4 + 7 = 11
```

En el ejemplo anterior:
- Copió solo 5 caracteres de src (espacio, m, u, n, d)
- Pero retorna 11 (que sería la longitud si hubiera copiado todo)

Esto permite detectar truncamiento:

```c
size_t result = ft_strlcat(dst, src, 10);

if (result >= 10) {
    // Se truncó: result=11, dstsize=10
    // Necesitarías un buffer de al menos 12 bytes (11 + '\0')
}
```

## Resumen del funcionamiento

1. Calcula longitudes de dst y src
2. Si dst ya está lleno o el buffer es inválido, retorna sin copiar
3. Copia caracteres de src a dst uno por uno
4. Se detiene cuando:
   - Termina src, O
   - La siguiente posición sería dstsize-1 (reservada para '\0')
5. Coloca '\0' donde se detuvo
6. Retorna la longitud total que hubiera tenido (dst_inicial + src completo)

El diseño garantiza que:
- Nunca sobrepasa dstsize
- dst siempre termina en '\0'
- El llamador puede saber si hubo truncamiento
