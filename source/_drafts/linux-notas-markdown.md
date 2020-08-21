---
title: 'Joplin: Una aplicación multiplaforma para gestionar notas' 
categories:
- Aplicaciones
- Linux
tags:
    - Linux
    - Software libre
thumbnail: /images/joplin/joplin-logo.png
date: 2020-04-24
layout: false
---

{% img /images/joplin/joplin-title.png "Migrando de Evernote a Joplin 'Migrando de Evernote a Joplin'"%}

Antes del confinamiento procedí a un cambio de mi entorno de trabajo y la transición completa a GNU/Linux. Poco a poco voy sustituyendo algunas de las herramientas por otras nuevas (preferentemente de software libre) para ayudar a gestionar las tareas y proyectos personales.
<!-- more -->

## Necesidad y requisitos

Trabajo con un equipo de sobremesa muy modesto en cuanto a prestaciones y creo que con buena relación calidad/precio. La distribución de GNU/Linux con la que llevo más de un año ya es [KDE Neon](https://neon.kde.org/) con uso diario. Por el momento cero problemas y muy a gusto.

Una de las necesidades que tenía era poder documentar muchos de los proyectos y las tareas de forma paralela a como lo hago con el trabajo. Toda esa información de interés que acaba en Pocket para consultar más tarde (casi nunca lo hago 😅) en Evernote, Notion, Google Docs, en Tweets guardados o simplemente en listas de texto desperdigadas

Quería que fuese una herramienta con la que poder **tomar notas de forma ágil y multiplataforma** pero sin adornos ni complejidad excesivos. Lo único que tenía claro era que el formato debería ser Markdown, reStructuredText o AsciiDoc.

La primera idea fue utilizar una estructura de archivos Markdown, utilizar Pandoc (que es espectacular) en caso de que necesitase convertir un documento, meterlo en un repositorio tipo Github, Gitlab o Bitbucket y listo.
Pues bien... esta idea no duro mucho, por varios motivos:

* Tenía que generar una estructura de archivos, agruparlos y mantenerlos
* Hasta la tarea más sencilla como renombrar implicaba la interacción con repositorios (que aunque  automatizable no era ágil)
* No era una solución multiplataforma por lo que en una tablet o móvil tendría que conectarme por SSH a una máquina para hacer los documentos
* Las cosas más comunes y simples, como adjuntar archivos o imágenes (capturas de pantalla) implicaban también muchas acciones lejos del copiar/pegar deseable
* No disponía de marcado con etiquetas para búsqueda
* No permitía la navegación referencial entre documentos

Con esto no quiero decir que un sistema de este tipo no funcione correctamente, pero buscaba algo mucho más directo.

Después de muchas pruebas con varias plataformas online topé con Joplin y la verdad es que me va a costar soltarlo.

## Joplin 

Joplin es lo más parecido a un reemplazo que he encontrado para Evernote/Notion y de paso para el resto de las herramientas que estaba utilizando. Como es de esperar no tiene parte de sus cosas *cool* que para mí son totalmente prescindibles en Evernote (OCR en imágenes, chat y demás) pero hace su función.

Su uso es para gestión de notas y to-do pero con algunas funcionalidades que lo hacen muy interesante:

* Permite la importación desde Evernote (formato .enex, aka Evernote XML)
* Permite Sincronización a/desde NextCloud, Dropbox, OneDrive, WebDAV o sistema de archivos (un volumen montado en red, por ejemplo)
* Se encuentra disponible para Windows, Linux, macOS, Android e iOS (la aplicación en terminal funciona en FreeBSD también)
* También es compatible con extensión para navegador Web Clipper, disponible para Chrome y Firefox para guardar páginas web y pantallazos desde el navegador directamente.

 {% img /images/joplin/joplin-overview.png "Página de inicio de Joplin 'Pantalla de inicio de Joplin en aplicación de escritorio'" %}
 
 ### El markdown y sus plugins
 
 Una de las primeras cosas que me apresuré a ver es el tipo de markdown (implementa la especificación [CommonMark](https://spec.commonmark.org/) y los plugins que soportaba, por lo menos para las cosas más habituales. La verdad es que me sorprendió. Tenía mucho más soporte del que me esperaba.
 
 La documentación del markdown soportado se puede encontrar [aquí](https://joplinapp.org/markdown/).
 
Los plugins actualmente soportados se pueden ver en la siguiente captura:
 
  {% img /images/joplin/joplin-markdown-plugins.png "Plugins de Joplin para Markdown 'Plugins de Joplin para Markdown'" %}

Seguro que muchos ya conocéis [mermaid](https://mermaid-js.github.io/mermaid), sino probadla incluso aunque no uséis Joplin porque es genial.

Aquí tenéis un pantallazo con algunos ejemplos de los plugins:
 {% img /images/joplin/joplin-mermaid.png "Ejemplos de plugins en Joplin 'Ejemplos de plugins en Joplin'" %}
 
 ### Web Clipper

Como ya he dicho se trata de una extensión que funciona con muchas plataformas además de Joplin. Permite crear y etiquetar notas desde el navegador.

### Sincronización con Nextcloud/Dropbox ...

Actualmente trabajo en una empresa que se dedica fundamentalmente a sistemas y para algún proyecto de envergadura hemos empezado a provisionar Nextcloud y lo utilizamos para la comunicación interna, videollamadas, etc.

La configuración de la sincronización es muy sencilla.

Si se va a sincronizar con un servicio remoto es importante activar la opción del cifrado de los datos.

(COMPLETAR)

### Para los amigos de la terminal

Joplin viene con alguna sorpresa para los amigos de la terminal: Una versión de consola, con la que podemos realizar las mismas tareas que con la versión de escritorio haciendo uso de la terminal (necesita tener instalado Node en versión 10+).

```
NPM_CONFIG_PREFIX=~/.joplin-bin npm install -g joplin
sudo ln -s ~/.joplin-bin/bin/joplin /usr/bin/joplin
```

