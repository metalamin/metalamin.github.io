---
title: "DTMF con algoritmo de Goertzel en ADSP-2181"
excerpt: "Detector de tonos DTMF con algoritmo de Goertzel en ADSP-2181"
header:
  teaser: "/assets/images/goertzel/teaser.png"
tags:
  - EN
  - Machform
  - SQL Injection
  - Path Traversal
  - RCE
hidden: true
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

El cálculo de todos los valor de la DFT no es necesario a la hora de implementar un detector DTMF, entonces hacer la FFT puede suponer un peso computacional innecesario y mejorable. Por lo que se hace uso del Algoritmo de Goertzel que permite calcular la DFT únicamente en las frecuencias deseadas para comprobar la presencia del par de tonos que corresponden al número marcado.

En este trabajo del laboratorio de tratamiento digital de la señal se pide implementar el citado Algoritmo de Goertzel que permite calcular un valor de X[k] mediante un ﬁltrado.

El objetivo del trabajo y de la ampliación es de obtener un detector funcional de marcación DTMF con una salida visual por el osciloscopio. La implementación se hace sobre la placa de desarrollo EZKIT-Lite de Analog Devices basada en el ADSP-2181 del mismo fabricante.
El algoritmo explicado en el documento del enunciado consiste en 2 partes: Una recursiva mientras se reciben datos, y la segunda que se hace cada N muestras para calcular el resultado de la DFT. 

En pseudo código tenemos que implementar lo siguiente:
```matlab
s_prev = 0;
s_prev2 = 0;
normalized_frequency = target_frequency / sample_rate;
coeff = 2 * cos (2* PI* normalized_frequency );
for each sample , x [ n ] , 
	s = x [ n ] + coeff ∗ s_prev − s_prev2 ; 
	s_prev2 = s_prev ; 
	s_prev = s ;
end 
power = s_prev2 ∗ s_prev2 + s_prev ∗ s_prev − coeff ∗ s_prev ∗ s_prev2 
```
Vemos que la segunda parte es un poco diferente a la propuesta en el trabajo ya que es una manera “optimizada” de obtener el resultado.
$$
magnitude^2= Q_1^2+Q_2^2-Q_1*Q_2^2*coef
$$