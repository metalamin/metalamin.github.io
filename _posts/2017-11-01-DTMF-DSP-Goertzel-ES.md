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
usemathjax: true
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

El cálculo de todos los valor de la DFT no es necesario a la hora de implementar un detector DTMF, entonces hacer la FFT puede suponer un peso computacional innecesario y mejorable. Por esa razón se hace uso del Algoritmo de Goertzel que permite calcular la DFT únicamente en las frecuencias deseadas para comprobar la presencia del par de tonos que corresponden al número marcado.

En este trabajo se pretende implementar el citado Algoritmo de Goertzel que permite calcular un valor de X[k] mediante ﬁltrado.

El es de obtener un detector funcional de marcación DTMF con una salida visual por el osciloscopio. La implementación se hace sobre la placa de desarrollo *EZKIT-Lite* de *Analog Devices* basada en el **ADSP-2181** del mismo fabricante.

El algoritmo consiste en 2 partes: Una recursiva mientras se reciben datos, y la segunda que se hace cada N muestras para calcular el resultado de la DFT. 

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
Vemos que la segunda parte es un poco diferente ya que es una manera “optimizada” de obtener el resultado.

$$
magnitude^2= Q_1^2+Q_2^2-Q_1*Q_2^2*coef
$$


# Trabajo previo
Para poder implementar el algoritmo de Goertzel en el DSP necesitamos calcular una serie de valores.


## Valor de N
**N** corresponde al numero de muestras que se hacen en la parte recursiva antes de hacer el cálculo ﬁnal del valor de la DFT. Fijamos una resolución del análisis espectral a 10Hz y haremos el calculo partiendo de este requerimiento. 

Siendo **k** los diferentes coeﬁcientes en los que se puede calcular la DFT. Tenemos: 

$$
\frac{f_{tono}}{k}=\frac{f_s}{N}
$$

Calculamos N para que el cambio de una unidad de k corresponda a 10Hz.

$$
N=\frac{f_s}{f_{tono}}=\frac{8000}{10}=800
$$

## Escalado de la señal de entrada

Para evitar saturación del ﬁltro necesitamos hacer un escalado de la señal de entrada. 

Después de hacer el análisis con tonos a la entrada, vemos que no es la manera correcta ya que saturaba. Eso es debido al sumatorio resultante de la parte recursiva cuyo valor máximo es el sumatorio de N valores cuyo valor absoluto máximo es 1. Como el valor de entrada del DSP esta normalizado entre -1 y 1, tendremos que hacer un escalado de 1/800. 

El valor a utilizar en el DSP es:
$$
ganancia=\frac{1}{800}*2^{15}=40.96
$$

Por lo tanto hay que realizar un escalado en el DSP de 40. De esta manera se evitar la saturación.

## Valores de los coeﬁcientes
Como se pretende detectar los 8 tonos de la tabla DTMF, tendremos que calcular los coeﬁcientes correspondientes.

$$
Coef=2*cos(2\pi*\frac{f_{tono}}{f_s})
$$

Para poder guardar los valores en coma ﬁja en el DSP queremos que tengan un valor absoluto inferior a la unidad. Calculamos el valor de coseno solo y ya lo multiplicaremos por 2 a posteriori. A continuación, tenemos la tabla de los coeficientes. (la mitad)

|Frecuencia|Coeficiente|Valor en el DSP|
|--|--|--|
|697 |0,8539 |27980 |
|770| 0,8226 |26956 |
|852| 0,7843 |25701 |
|941| 0,7391 |24219 |
|1209| 0,5821 |19073|
|1336| 0,4982 |16325 |
1477 |0,3993 |13085 |
1633 |0,2843 |9315|


## Amplitud de detección de tono 

El valor del tono detectado tiene que ser superior al 20% de la amplitud máxima de entrada para considerarse positivo. 
Tenemos que el valor máximo a la entrada es de 40 (en el DSP) por el escalado. Por lo que ﬁjaremos el umbral al 20% de 40
$$
umbral=40*20/100=8
$$
Se considerará el tono detectado cuando supere ese umbral.

# Programa en MATLAB
Antes de empezar a programar en el DSP se trabaja en MATLAB para veriﬁcar el funcionamiento correcto del algoritmo. 

El archivo correspondiente de MATLAB es [migoertzel.m](https://github.com/metalamin/DSP-Goertzel/blob/master/migoertzel.m) e implementa la función:

$$
y = migoertzel(x)
$$

El programa primero inicializa los valores y luego ejecuta 2 bucles: Uno para hacer ventanas de 800 muestras y luego el bucle correspondiente a la primera parte del algoritmo. Finalmente calcula el valor ﬁnal para todos los tonos. 

No saca el valor del dígito marcado pero se puede apreciar fácilmente en que momentos se supera el umbral de los tonos. 

Sea x la señal muestreada a 8000Hz se usa de la siguiente manera:
```matlab
>> x= sin (2* pi * t *770) ; % 770Hz
>> y= migoertzel (x /800) *2^15; 
>> round (y ) 
ans = 0 8132 0 0 0 0 0 0
```

Vemos que se ha dividido la señal por N para hacer el escalado que tendríamos que hacer en el DSP. En este ejemplo se aprecia como detecta perfectamente el segundo tono correspondiente a la frecuencia 770 Hz. Si se hace con un tono puro y amplitud máxima de entrada vemos que los valores son son próximos a 8000, superando con varios ordenes de magnitud el umbral. 

Probamos a ver si con una frecuencia cercana da un falso positivo.

```matlab
>> x= sin (2* pi * t *760) ; % 760Hz
>> y= migoertzel (x /800) *2^15; 
>> round (y ) 
ans = 0 0 0 0 0 0 0 0
```

No se detecta la frecuencia en este caso. Por lo que se comporta como es deseado.

# Programa en MATLAB
Partimos de la simple detección de la marcación del 0 que enciende un led, luego se amplia para detectar los 8 tonos y sacar por el osciloscopio una respuesta que caracteriza cada numero. 

## Detector de marcación del Cero.

Esta primera parte consigue la detección de 2 tonos correspondientes al ’0’. Para ello implementa el algoritmo de Goertzel con buffer circular para ir haciendo la parte recursiva. Luego se repite el mismo código para cada tono (2 veces).


```matlab
mx0=dm( i2 ,m2); 
my0=dm( coef1 );	{Cargamos q1 y coef } 
mr=mx0*my0( ss );	{q1* cos ( alpha ) }

my0=1; 
mr=mr1*my0( ss );	{q1 *2* cos ( alpha ) } 
ar=mr0;

mx0 = dm( rx_buf + 1); { input } 
my0=dm( ganancia );
mr=mx0*my0( ss ); 
ay0=mr1; 
ar=ar+ay0;			{ input+q1 *2* cos ( alpha ) } 
ay0=dm( i2 ,m3);	{cargamos q2}

ar=ar−ay0;			{ input+q1 *2* cos ( alpha )−q2} 
dm( i2 ,m3)=ar ;
``` 

Si queremos ver si funciona bien sacamos por la pantalla los valores de q1 o de q2 por el osciloscopio cada muestra. Y debería dar algo parecido a la siguiente ﬁgura cuando en la entrada se introduce el tono correspondiente.

<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/DTMF-DSP/DSP.jpg" alt="">
  <figcaption style="text-align: center">Señal correspondiente a los valores de q1</figcaption>
</figure>

Vemos que el valor de q1 se va haciendo mas grande hasta llegar a las N muestras. Entonces se usan los valores y se vuelve a reinicializar para dejarlo preparado para la siguiente pasada de N muestras.

Después de N muestras se hace el calculo ﬁnal.
```matlab
ar=dm( coef1 ); 
my0=1; mr=ar ∗my0( ss ); 
ar=mr0;				{ coef * 2}

mx0=dm( i2 ,m2);	{Obtener q1 dos veces} 
my0=mx0; 
mx1=dm( i2 ,m2);	{Obtener q2 dos veces} 
my1=mx1;
mr=0; mf=mx0*my1( ss );	{q1*q2} 
mr=mr − ar ∗mf( ss );	{−q1*q2* coef} 
mr=mr+mx0*my0( ss );	{q1^1−q1*q2* coef}
mr=mr+mx1*my1( ss );	{q2^2+q1^2−q1*q2* coef} 
dm( sqr1 )=mr1;
```

