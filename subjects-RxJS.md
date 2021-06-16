# Subjects RxJS

slides: https://docs.google.com/presentation/d/1-aPERvhu1w1_jG0RIBWVrwG34xtHm3nBoV-EMxM9gk4/edit?usp=sharing


Entendiendo los Observables.

A veces el funcionamiento o la propia naturaleza de los Obseravables es díficil de entender porque sencillamente las cosas no funcionan como esperamos. Un buen ejemplo de esto es que es habitual pensar que si tenemos un único Observable y distintas subscripciones a ese mismo Observable, todas y cada una de ellas recibirán el mismo valor. Tiene sentido, ¿verdad?. Veamos si esto es realmente así:

```
const myObservable = Rx.Observable.create((observer) => {
    observer.next(Math.random()); // generamos un número aleatorio
});

// Subscripción 1
myObservable.subscribe(data => {
  console.log(data); // 0.799234057357979
});

// Subscripción 2
myObservable.subscribe(data => {
   console.log(data); //0.9293311109721503
});

```
Como podemos ver, en cada una de las subscripciones estamos recibiendo valores distintos porque por defecto los Observables no realizan ningún tipo de "sincronización" entre las distintas subscripciones que escuchan, ya que realmente, éstas, reciben su propio stream de datos y para explicar esto, es necesario entender los dos tipos principales de Observables que existen:


Observables fríos

Los Observables "fríos" son aquellos que no emiten valores hasta que haya una suscripción activa, ya que la información es producida dentro del Observable y por tanto solo emiten valores en el momento en que se establece una nueva subscripción, por eso, el ejemplo previo que hemos visto, math.random() devuelve valores diferentes.


Observables calientes

Por contra, los Observables "calientes" son aquellos que pueden emitir valores sin que haya ninguna subscripción activa, porque la información del stream se produce fuera del propio Observable. RxJs dispone de algunos Observables ¨calientes¨ y el mejor ejemplo de éstos, es fromEvent que nos permite establecer un Observable sobre cualquier tipo de evento como el click del ratón:

```
const myObservable = Rx.Observable.fromEvent(document, 'click'); // Escuchamos el evento click

// Subscripción 1
observable.subscribe(e => {
  console.log(e.clientX); // 408
});

// Subscripción 2
observable.subscribe(e => {
   console.log(e.clientX); // 408
});

```
Como vemos en el ejemplo, ahora si recibimos en ambas subscripciones el mismo valor, porque fromEvent es un Observable "caliente" (la información se produce dentro del Observable) y no necesita que haya ninguna subscripción activa para emitir sus valores, pero ¿Qué hacemos cuando queremos crear un Observable y compartir el mismo stream con todas las subscripciones sin preocuparnos de si el Observable es frío o caliente? ¡Subjects al rescate!.


Subjects

Los subjects de RxJs son un tipo de Observable especial que nos permiten realizar diversas tareas como el multicasting, es decir, compartir exactamente el mismo stream de datos con todas las subscripciones sin preocuparnos del tipo de Observable que estamos manejando.

Aparte, hay otra característica de los Subjects que les da una gran versatilidad y es que los Subjects de RxJs son Observables y Observers al mismo tiempo por lo que nos podemos subscribir a un Subject como a cualquier otro Observable, pero además disponen de los métodos next(), error() y complete() que tienen el Observer para emitir sus valores.

Vamos a volver a reproducir el primer ejemplo usando Subjects:

```
const subject = new Rx.Subject();// creamos nuestro subject

// Subscripción 1 al Subject que es un Observable
subject.subscribe((data) => {
    console.log(data); // 0.799234057357979
});

// Subscripción 2 al Subject que es un Observable
subject.subscribe((data) => {
    console.log(data); // 0.799234057357979
});

subject.next(Math.random());// El subject usa el método next para emitir valores ya que también es un Observer.

```
Como vemos, ahora sí, gracias al multicasting que realizan los Subjects, tenemos la plena seguridad de que todas las subscripciones reciben exactamente el mismo valor sin importar si el Observable es frío o caliente.

RxJs dispone de diferentes tipos de Subjects que vienen a cubrir distintas necesidades como por ejemplo ¨recordar¨ el último valor emitido por el observable cuando se establece una nueva subscripción:


Behaviour Subject

Behaviour Subject nos permite utilizar una característica realmente útil y que es la de poder "recodar¨ el último valor emitido por el Observable a todas las nuevas subscripciones, al margen del momento temporal en que éstas se establezcan, actuando como un mencanismo de "sincronización" entre todas las subscripciones que resulta realmente últil:



```
const subject = new Rx.BehaviorSubject(5); //Valor de iniciación

// Subscripción 1
subject.subscribe(data => {
    console.log('Subscripción 1:', data);
});

subject.next(1);
subject.next(2);

// Subscripción 2
subject.subscribe(data => {
    console.log('Subscripción 2:', data);
});

subject.next(3);
/*
Subscripción 1: 5
Subscripción 1: 1
Subscripción 1: 2
Subscripción 2: 2 La segunda subscripción recibe el último valor emitido
Subscripción 1: 3
Subscripción 2: 3
*/

```
Como podemos ver en la salida, se estable una subscripción, se emite el valor inicial y los valores 1 y 2, y cuando la segunda subscripción se establece, ésta recibe el último valor emitido por el Observable, es decir el 2.

Behaviour Subject es probablemente el Subject más usado, ya que es la base de la mayoría de implementaciones de Redux basadas en RxJs como @ngRxStore para Angular o una multiplataforma que escribí recientemente, llamada Rextore y que podéis encontrar en mi github. Behaviour Subject, además de recordar el último valor emitido a todas las subscripciones, nos permite definir un valor por defecto, que sería el equivalente al initial state de Redux:

```
const subject = new Rx.BehaviorSubject('Hello');// Hello es el valor por defecto del Observable

subject.subscribe(data => {
    console.log(data); //Hello
});

```

Replay Subject

Replay Subject funciona de la misma forma que Behaviour Subject, pero así como Behaviour Subject solo tiene la habilidad de recordar el último valor emitido, con Replay Subject vamos a poder configurar el número de valores que queremos recordar a las nuevas subscripciones:



```
const subject = new Rx.ReplaySubject(2); // Indicamos que queremos ¨recordar¨ los dos últimos valores

// Subscripción 1
subject.subscribe(data => {
    console.log('Subscripción 1:', data);
});

subject.next(1);
subject.next(2);

// Subscripción 2
subject.subscribe(data => {
    console.log('Subscripción 2:', data);
});

subject.next(3);
/*
Subscripción 1: 1
Subscripción 1: 2
Subscripción 2: 1 La segunda subscripción recibe el penultimo valor emitido
Subscripción 2: 2 La segunda subscripción recibe el último valor emitido
Subscripción 1: 3
Subscripción 2: 3
*/

```

Async Subject

Async subject es el tercer tipo de Subjects de RxJs y tiene una particularidad bastante interesante pero que seguramente nos costará un poquito encontrar un uso para él, ya que Async Subject solo emitirá el último valor del Observable cuando éste haya finalizado, es decir, cuando se haya ejecutado el método complete():



```
const subject = new Rx.AsyncSubject();

// Subscripción 1
subject.subscribe(data => {
    console.log('Subscripción 1:', data);
});

subject.next(1);
subject.next(2);

// Subscripción 2
subject.subscribe(data => {
    console.log('Subscripción 2:', data);
});

subject.next(3);
subject.complete(); // Finalizamos el Observable
/*
Subscripción 1: 3
Subscripción 2: 3
*/

```
