---
title: "Recuperar un Perl desde un EXE"
excerpt: "Como recuperar Perl empaquetado en EXE con PerlApp de ActiveState "
header:
  teaser: "/assets/images/2018-03-06-Liberando-el-patinete-Xiaomi-m365-Parte-1-La-App/fotopatinete2.jpg"
categories:
  - ES
tags:
  - ES
  - OllyDbg
  - Perl
  - Reversing

---
Recientemente, durante un pentest en mis labores como patata ninja, me he encontrado con un Perl en forma de ejecutable de Windows. Éste sirve para hacer una serie de operaciones contra el dominio y para poder autenticar parece que genera en memoria las credenciales desde un archivo de claves cifrado. ¿Cómo puedo recuperar esas credenciales?


Una manera podría haber sido montar un controlador de dominio falso estilo Responder. Pero… ¿Por qué hacerlo fácil? Además, tenía curiosidad en averiguar que método de cifrado se usa, por si me volvía a encontrar algo parecido más adelante durante el pentest.

## Análisis del ejecutable
Un breve análisis con strings nos muestra que se trata de un script en Perl empaquetado en un ejecutable EXE. En concreto, parece que se utilizó el PerlApp de ActiveState para tal fin.
Hasta el momento todos los packers de Perl a EXE que me he encontrado tienen que guardar en claro el script antes de poder lanzar el interpretador. En algunos casos lo guardan en un archivo y en otros se quedan en memoria. En la muestra que nos interesa lo desempaqueta en memoria. Vamos a ver una menara de recuperarlo usando el OllyDbg.

