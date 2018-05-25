---
title: "Recuperar un Perl desde un EXE (ES)"
excerpt: "Como recuperar Perl empaquetado en EXE con PerlApp de ActiveState "
header:
  teaser: "/assets/images/Perl-from-EXE/teaser_es.png"
tags:
  - ES
  - OllyDbg
  - Perl
  - Reversing
gallery1:
          - url: /assets/images/Perl-from-EXE/referenced.png
            image_path: /assets/images/Perl-from-EXE/referenced.png
            alt: "Referenced strings"
            title: "Referenced strings"
          - url: /assets/images/Perl-from-EXE/strings_2.png
            image_path: /assets/images/Perl-from-EXE/strings_2.png
            alt: "Strings"
            title: "Strings"
---
Hace tiempo, durante uno de mis ejercicios de Red Team, me encontré con un Perl en forma de ejecutable de Windows. Éste sirve para hacer una serie de operaciones contra el dominio y para poder autenticar parece que genera en memoria las credenciales desde un archivo de claves cifrado. ¿Cómo puedo recuperar esas credenciales?


Una manera podría haber sido montar un controlador de dominio falso estilo [Responder](https://github.com/SpiderLabs/Responder). Pero… ¿Por qué hacerlo fácil? Además, tenía curiosidad en averiguar que método de cifrado se usa, por si me volvía a encontrar algo parecido más adelante durante el pentest.

## Análisis del ejecutable
Un breve análisis con strings nos muestra que se trata de un script en Perl empaquetado en un ejecutable EXE. En concreto, parece que se utilizó el **PerlApp de ActiveState** para tal fin.

<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/strings.png" alt="">
  <figcaption style="text-align: center">strings decode.exe | grep -i perl</figcaption>
</figure> 


Hasta el momento todos los packers de Perl a EXE que me he encontrado tienen que guardar en claro el script antes de poder lanzar el interpretador. En algunos casos lo guardan en un archivo y en otros se quedan en memoria. En la muestra que nos interesa lo desempaqueta en memoria. Vamos a ver una menara de recuperarlo usando el [OllyDbg](http://www.ollydbg.de/).

## Recuperando el Perl
Cargamos el ejecutable en el OllyDbg. Pero antes de lanzar la ejecución vamos a poner un breakpoint en algún punto después de la carga del script.
Para ello primero mostramos las cadenas de texto.
{% include gallery id="gallery1" %}
Sabemos que después de desempaquetar el script, el ejecutable tiene que montar la cadena para la llamada del interprete Perl. Vamos a buscar las cadenas que parezcan argumentos para el lanzamiento del interprete de Perl y parar la ejecución en ese punto. Elegimos algunas de las cadenas y fijamos el breakpoint apretando **F2**.

<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/strings_4.png" alt="">
  <figcaption style="text-align: center">Strings de parámetros</figcaption>
</figure>

Ya estamos listos para arrancar el programa apretando **F9**.

<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/stop.png" alt="">
  <figcaption style="text-align: center">Ejecución parada</figcaption>
</figure>

Una vez parado en nuestro breakpoint, abrimos el Memory Map pulsando **"Alt+M"**.
<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/memory.png" alt="">
  <figcaption style="text-align: center">Memory Map</figcaption>
</figure>

Con la ventana de memoria abierta hacemos una búsqueda (**Ctrl+B**) de cadenas que puedan contener el script que buscamos. Por ejemplo, las llamadas a las librerías en perl que empiezan por **"use"** (nótese el espacio al final).
<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/busqueda.png" alt="">
  <figcaption style="text-align: center">Búsqueda de cadenas</figcaption>
</figure>

Ya obtenemos el script en memoria y sabemos donde se encuentra. Tan solo queda guardar ese segmento de memoria en un archivo y quedarnos con el script.
<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/backup.png" alt="">
  <figcaption style="text-align: center">Backup</figcaption>
</figure>

Y con esto hemos recuperado el script entero incluidos los comentarios del programador.
<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/recovered.png" alt="">
  <figcaption style="text-align: center">Script Perl recuperado</figcaption>
</figure>

 

Seguramente podríamos haberlo hecho de manera más elegante o más rápida, pero ésta es la que se me ocurrió en el momento y quise compartir.