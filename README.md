# README — Notas técnicas y mejoras aplicadas al servidor TCP

Este documento resume las decisiones técnicas, correcciones y mejoras aplicadas al proyecto del servidor TCP multihilo encargado de registrar usuarios, iniciar sesión, listar datos, generar hashes y manejar múltiples clientes simultáneos. Incluye advertencias, problemas detectados y justificación de todas las mejoras implementadas.

SECCIÓN 1 — Corrección en UserDataThread.java
=
Problema original:
```System.exit(0);```  
Motivo del problema:
- Finaliza todo el proceso del servidor.
- Mata todos los hilos, incluso los que atienden a otros clientes.

Solución aplicada:
- Sustituido por ```return;```

Beneficio:
- Solo termina el hilo actual.
- El servidor sigue funcionando normalmente.


SECCIÓN 2 — Corrección en UserRepository.java
=

Versión original del método getAll:
```java
@Override
synchronized public List<T> getAll() {
    return (List<T>) userList;
}
```

Problema:
- Retorna la lista interna del repositorio.
- Otro hilo podría modificarla mientras otro la está iterando.
- Riesgo real de ConcurrentModificationException.
- Expones el recurso crítico ⇒ rompes encapsulación.

Solución aplicada:
```java
@Override
synchronized public List<T> getAll() {
    return new ArrayList<>((List<T>) userList);
}
```

Beneficios:
- Los hilos reciben una copia segura.
- Evita corrupción del estado compartido.
- Mantiene intacta la zona crítica del repositorio.

SECCIÓN 3 — Mejora en GetHashUseCase
=

Problema detectado:
- Comando hardcodeado a macOS: ```md5```
- No funcionaría en Windows o Linux sin adaptaciones.

Ideas para mejora futura:
- Detectar SO con System.getProperty("os.name")
- Elegir automáticamente:
  - Windows → CertUtil -hashfile file MD5
  - Linux → md5sum file
  - macOS → md5 file

(Notas: en examen no es obligatorio implementarlo.)

SECCIÓN 4 — Observación en LoginUseCase
=

Problema potencial:
- Email podría estar duplicado en la lista.
- Generaría ambigüedad al hacer login.

Motivo por el que no se corrige:
- Para examen se asume que el repositorio NO tendrá duplicados.
- Aun así se deja anotado como consideración técnica.

SECCIÓN 5 — Corrección en RegisterUseCase
=

Problema original:
Path hardcodeado a tu máquina:
```java
final String path = "recuperacion_examen_final_socket_marzo_23/files/";
```

Motivo del problema:
- Si el directorio no existe ⇒ el registro falla.
- Dependencia innecesaria del entorno físico del profesor/alumno.

Solución aplicada:
- Se usa un path relativo estable dentro del proyecto:
```java
final String path = "files/";
```

Notas:
- La carpeta "files" SIEMPRE existe en el entregable del examen.
- No se elimina compatibilidad ni estructura original.


SECCIÓN 6 — Explicación de Routing y sus endpoints
=
El constructor de Routing registra todos los comandos disponibles:

(reg) → RegisterUseCase  
(log) → LoginUseCase  
(datu) → UserDataUseCase  
(list) → UsersListUseCase  
(hash5) → GetHashUseCase  
(fin) → LogoutUseCase  

Código original explicado con comentarios (solo descripción, no cambiar):

```java
public Routing() {
    this.repository = new UserRepository<>();
    maganerEndPoints = new HashMap<>();

    maganerEndPoints.put("reg",  new RegisterUseCase(this.repository));   // Registro
    maganerEndPoints.put("log",  new LoginUseCase(this.repository));     // Login
    maganerEndPoints.put("datu", new UserDataUseCase(this.repository));  // Datos usuario
    maganerEndPoints.put("list", new UsersListUseCase(this.repository)); // Listar usuarios
    maganerEndPoints.put("hash5", new GetHashUseCase(this.repository));  // Hash MD5
    maganerEndPoints.put("fin",  new LogoutUseCase(this.repository));    // Logout
}
```
## Captura de Servidor con varios CLientes (solucionado el problema de forzar desconexion a todos los clientes si uno se desconecta)
<img width="2242" height="701" alt="imagen" src="https://github.com/user-attachments/assets/0f2edcd5-0a06-4e63-927f-7205e2db3b12" />
## Captura de Servidor y Cliente para ver mejor la interfaz
<img width="2143" height="664" alt="imagen" src="https://github.com/user-attachments/assets/1adf5563-e7a3-4b83-ac27-bf231311c056" />
## Captura de Cliente con todos los comandos testeados y funcionando perfectamente
<img width="1807" height="889" alt="imagen" src="https://github.com/user-attachments/assets/fd50c6f3-2b89-46a0-8f69-588a13f860d8" />


