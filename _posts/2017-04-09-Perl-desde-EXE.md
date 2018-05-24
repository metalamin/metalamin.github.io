---
title: "Recuperar un Perl desde un EXE"
excerpt: "Como recuperar Perl empaquetado en EXE con PerlApp de ActiveState "
header:
  teaser: "/assets/images/Perl-from-EXE/teaser.png"
tags:
  - ES
  - OllyDbg
  - Perl
  - Reversing

---
Hace tiempo, durante uno de mi de Red Team, me he encontrado con un Perl en forma de ejecutable de Windows. Éste sirve para hacer una serie de operaciones contra el dominio y para poder autenticar parece que genera en memoria las credenciales desde un archivo de claves cifrado. ¿Cómo puedo recuperar esas credenciales?


Una manera podría haber sido montar un controlador de dominio falso estilo Responder. Pero… ¿Por qué hacerlo fácil? Además, tenía curiosidad en averiguar que método de cifrado se usa, por si me volvía a encontrar algo parecido más adelante durante el pentest.

## Análisis del ejecutable
Un breve análisis con strings nos muestra que se trata de un script en Perl empaquetado en un ejecutable EXE. En concreto, parece que se utilizó el PerlApp de ActiveState para tal fin.

strings decode.exe | grep -i perl

<figure style="width: auto" class="align-center">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/strings.png" alt="">
  <figcaption>strings decode.exe | grep -i perl</figcaption>
</figure> 


Hasta el momento todos los packers de Perl a EXE que me he encontrado tienen que guardar en claro el script antes de poder lanzar el interpretador. En algunos casos lo guardan en un archivo y en otros se quedan en memoria. En la muestra que nos interesa lo desempaqueta en memoria. Vamos a ver una menara de recuperarlo usando el OllyDbg.

## Recuperando el Perl
Cargamos el ejecutable en el OllyDbg. Pero antes de lanzar la ejecución vamos a poner un breakpoint en algún punto después de la carga del script.

Para ello primero mostramos las cadenas de texto.

Referenced strings
Referenced strings
Strings
Strings
Sabemos que después de desempaquetar el script, el ejecutable tiene que montar la cadena para la llamada del interprete Perl. Vamos a buscar las cadenas que parezcan argumentos para el lanzamiento del interprete de Perl y parar la ejecución en ese punto. Elegimos algunas de las cadenas y fijamos el breakpoint apretando F2.

Strings de parámetros
Strings de parámetros
Ya estamos listos para arrancar el programa apretando F9.
Ejecución parada
Ejecución parada
Una vez parado en nuestro breakpoint, abrimos el Memory Map pulsando “Alt+M“.
Memory Map
Memory Map
Con la ventana de memoria abierta hacemos una búsqueda (Ctrl+B) de cadenas que puedan contener el script que buscamos. Por ejemplo, las llamadas a las librerías en perl que empiezan por “use “ (nótese el espacio al final).
Búsqueda de cadenas
Búsqueda de cadenas
Ya obtenemos el script en memoria y sabemos donde se encuentra. Tan solo queda guardar ese segmento de memoria en un archivo y quedarnos con el script.
Backup
Backup
Y con esto hemos recuperado el script entero incluidos los comentarios del programador.
Script Perl recuperado
Script Perl recuperado
 

Seguramente podríamos haberlo hecho de manera más elegante o más rápida, pero ésta es la que se me ocurrió en el momento y quise compartir.