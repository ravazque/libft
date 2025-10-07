# 📚 README - ft_strlcat

## 🎯 Propósito de la Función

`ft_strlcat` es una función que concatena (une) dos strings de forma **segura**, evitando desbordamientos de buffer. Es una reimplementación de la función `strlcat` de BSD.

---

## 📋 Prototipo

```c
size_t ft_strlcat(char *dst, const char *src, size_t dstsize)
```

### Parámetros:
- **dst**: String destino donde se concatenará src
- **src**: String fuente que se añadirá a dst
- **dstsize**: Tamaño total del buffer dst (NO el espacio disponible)

### Retorno:
- La longitud total que **intentó** crear (longitud de dst + longitud de src)

---

## 🔑 Concepto Clave: ¿Por qué no devuelve lo que hace?

### Esta es la parte más confusa pero más importante:

**Lo que HACE la función:**
- Concatena src al final de dst (copia caracteres)
- Se asegura de que dst termine con '\0'
- NO sobrepasa el límite de dstsize

**Lo que DEVUELVE la función:**
- La longitud TOTAL que hubiera tenido el string si hubiera espacio suficiente
- Es decir: `strlen(dst_inicial) + strlen(src)`

### ¿Por qué esta diferencia?

Esto permite **detectar truncamiento**:
```c
size_t resultado = ft_strlcat(dst, src, size);

if (resultado >= size) {
    // ¡ALERTA! El string fue truncado
    // No cupo todo src en dst
}
```

---

## 🔍 Análisis Paso a Paso

### Paso 1: Calcular longitudes
```c
lngdst = ft_strlen(dst);  // Longitud actual de dst
lngsrc = ft_strlen(src);  // Longitud de src
i = 0;                     // Índice para recorrer src
```

### Paso 2: Caso especial - Buffer demasiado pequeño
```c
if (lngdst >= dstsize)
    return (dstsize + lngsrc);
```

**¿Qué significa esto?**
- Si dst ya es igual o mayor que dstsize, el buffer está mal configurado o ya lleno
- Retorna `dstsize + lngsrc` como si dst tuviera longitud dstsize
- **NO COPIA NADA** en este caso

**Ejemplo:**
```
dst = "Hola mundo completo" (19 chars)
dstsize = 10
src = "extra"

lngdst (19) >= dstsize (10) → Retorna 10 + 5 = 15
```

### Paso 3: Copiar caracteres de src a dst
```c
while (src[i] != '\0' && (lngdst + i) < (dstsize - 1))
{
    dst[lngdst + i] = src[i];
    i++;
}
```

**Condiciones del while:**
1. `src[i] != '\0'` → No hemos llegado al final de src
2. `(lngdst + i) < (dstsize - 1)` → Hay espacio para este char + '\0'

**¿Por qué `dstsize - 1`?**
- Necesitamos reservar un espacio para el '\0' final
- Si dstsize = 10, solo podemos escribir hasta la posición 8 (índice 9 es para '\0')

**Visualización:**
```
dst antes:  "Hola\0?????"  (dstsize = 10)
            [0][1][2][3][4][5][6][7][8][9]
lngdst = 4

Copiando " mundo":
- i=0: dst[4] = ' '  → "Hola \0????"
- i=1: dst[5] = 'm'  → "Hola m\0???"
- i=2: dst[6] = 'u'  → "Hola mu\0??"
- i=3: dst[7] = 'n'  → "Hola mun\0?"
- i=4: dst[8] = 'd'  → "Hola mund\0"
- i=5: STOP (lngdst + i = 9, que NO es < 9)
```

### Paso 4: Añadir el terminador nulo
```c
dst[lngdst + i] = '\0';
```

Garantiza que dst siempre sea un string válido terminado en '\0'.

### Paso 5: Retornar la longitud intentada
```c
return (lngdst + lngsrc);
```

**Retorna SIEMPRE:**
- Longitud de dst original + longitud completa de src
- **Independientemente** de cuánto se copió realmente

---

## 📊 Ejemplos Completos

### Ejemplo 1: Concatenación exitosa (todo cabe)

```c
char dst[20] = "Hola";        // lngdst = 4
char src[] = " mundo";        // lngsrc = 6
size_t result;

result = ft_strlcat(dst, src, 20);

// RESULTADO:
// dst = "Hola mundo"
// result = 10 (4 + 6)
// result < 20 → ¡TODO CUPÓ!
```

### Ejemplo 2: Truncamiento (no todo cabe)

```c
char dst[10] = "Hola";        // lngdst = 4
char src[] = " mundo!";       // lngsrc = 7
size_t result;

result = ft_strlcat(dst, src, 10);

// RESULTADO:
// dst = "Hola mun\0"   (solo copió 4 chars de src + '\0')
// result = 11 (4 + 7)
// result >= 10 → ¡SE TRUNCÓ!
```

### Ejemplo 3: Buffer inválido

```c
char dst[5] = "Hola mundo";  // ¡Mal! dst dice tener más de 5
char src[] = "extra";         // lngsrc = 5
size_t result;

result = ft_strlcat(dst, src, 5);

// RESULTADO:
// dst = NO SE MODIFICA
// result = 10 (5 + 5)
// Detecta que algo está mal
```

---

## ⚠️ Diferencias con strcat

| Aspecto | strcat | ft_strlcat |
|---------|--------|------------|
| **Seguridad** | Insegura, puede desbordarse | Segura, respeta límites |
| **Parámetros** | Solo dst y src | dst, src y dstsize |
| **Retorno** | Puntero a dst | Longitud total intentada |
| **Detecta truncamiento** | No | Sí (result >= dstsize) |
| **Siempre añade '\0'** | Sí | Sí |

---

## 🧠 Lógica Interna Resumida

1. **Mide** las longitudes de dst y src
2. **Verifica** que dst no esté ya lleno (lngdst < dstsize)
3. **Copia** caracteres de src a dst hasta:
   - Terminar src, O
   - Quedarse sin espacio (dstsize - 1)
4. **Añade** '\0' al final de dst
5. **Retorna** lngdst + lngsrc (lo que "quiso" crear)

---

## 🎓 ¿Por qué esta función es así?

### Filosofía de diseño:

1. **Seguridad primero**: Nunca desborda el buffer
2. **Información útil**: El retorno permite detectar problemas
3. **Consistencia**: Siempre termina dst con '\0'
4. **Simplicidad**: Una sola función para concatenar con seguridad

### La clave está en el retorno:

```c
size_t needed = ft_strlcat(dst, src, size);

if (needed < size) {
    // ✅ Todo bien, concatenación completa
} else {
    // ❌ Se truncó, necesitábamos un buffer de tamaño 'needed + 1'
    printf("Necesitabas al menos %zu bytes\n", needed + 1);
}
```

---

## 💡 Casos de Uso Comunes

### 1. Construir rutas de archivos
```c
char path[256] = "/home/user";
ft_strlcat(path, "/documents", 256);
ft_strlcat(path, "/file.txt", 256);
// path = "/home/user/documents/file.txt"
```

### 2. Crear mensajes
```c
char msg[100] = "Error: ";
ft_strlcat(msg, error_type, 100);
ft_strlcat(msg, " en línea ", 100);
ft_strlcat(msg, line_number, 100);
```

### 3. Validar espacio necesario
```c
char buffer[50];
strcpy(buffer, "Inicio");

if (ft_strlcat(buffer, largo_string, 50) >= 50) {
    printf("Buffer demasiado pequeño\n");
}
```

---

## 🐛 Errores Comunes

### Error 1: Confundir dstsize con espacio disponible
```c
❌ ft_strlcat(dst, src, strlen(dst));  // ¡Mal! No hay espacio para src
✅ ft_strlcat(dst, src, sizeof(dst));  // Bien, tamaño total del buffer
```

### Error 2: No verificar el retorno
```c
❌ ft_strlcat(dst, src, size);  // Ignora posible truncamiento
✅ if (ft_strlcat(dst, src, size) >= size) {
    // Manejar truncamiento
}
```

### Error 3: Usar con buffers no inicializados
```c
❌ char dst[50];
   ft_strlcat(dst, "texto", 50);  // dst no tiene '\0' inicial

✅ char dst[50] = "";  // o dst[0] = '\0'
   ft_strlcat(dst, "texto", 50);
```

---

## ✅ Checklist de Comprensión

- [ ] Entiendo que dstsize es el tamaño TOTAL del buffer
- [ ] Sé que el retorno es lngdst + lngsrc (no lo que copió)
- [ ] Puedo detectar truncamiento comparando retorno con dstsize
- [ ] Comprendo por qué se usa `dstsize - 1` en el while
- [ ] Sé que dst siempre quedará con '\0' al final
- [ ] Entiendo el caso especial cuando lngdst >= dstsize

---

## 🔗 Recursos Adicionales

- Man page: `man strlcat` (en sistemas BSD)
- [OpenBSD strlcat](https://man.openbsd.org/strlcat.3)
- Norminette 42: Estilo de código de 42