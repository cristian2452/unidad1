# Patrones de diseño

## Tabla de contenidos:
- Introducción
- Singleton
- Iterador
- Adaptador
- Factory Method
- Observador
- Referencias bibliográficas


## Introducción
Los patrones de diseño son unas técnicas para resolver problemas comunes en el desarrollo de software y otros ámbitos referentes al
diseño de interacción o interfaces.

Un patrón de diseño resulta ser una solución a un problema de diseño. Para que una solución sea considerada un patrón debe poseer 
ciertas características. Una de ellas es que debe haber comprobado su efectividad resolviendo problemas similares en ocasiones 
anteriores. Otra es que debe ser reutilizable, lo que significa que es aplicable a diferentes problemas de diseño en distintas 
circunstancias.

## Singleton

Los objetos residen dentro de la memoria, y podemos instanciar tantos objetos como lo permita el espacio físico en la memoria.
Pero, en algunos casos, podemos tener una situación en la que solo se puede instanciar una instancia de una clase. Entonces,
imagina que estamos desarrollando un programa que está reproduciendo archivos de audio. Dentro de ese programa, necesitamos tener una 
clase que maneje la salida de audio. Una computadora generalmente tiene una salida de audio, por lo que no se puede reproducir más de 
un sonido a la vez. Por lo tanto, una clase que maneje el dispositivo de audio de la computadora debe tener exactamente una instancia.

¿Cómo podemos asegurarnos de que solo se crea una instancia? Cada clase java tiene un constructor público predeterminado, que puede 
invocarse desde cualquier parte del código. Si implementamos una clase donde el constructor predeterminado tiene un alcance "privado", 
entonces solo los métodos de esa clase pueden invocar ese constructor, lo que significa que no podemos instanciar esa clase de otras 
clases. Esta es una base del patrón Singleton.

Singleton garantiza que solo se puede crear un objeto (único) a partir de la clase.

![imagen](http://www.design-patterns-stories.com/assets/img/uml/singleton.png)

### Implementcion

**Singleton.java**

~~~~
package com.hundredwordsgof.singleton;

public class Singleton {

  private static Singleton INSTANCE;

  private Singleton() {
  }

  public static Singleton getInstance() {
    if (INSTANCE == null) {
      INSTANCE = new Singleton();
    }
    return INSTANCE;
  }
}
~~~~

### uso

**SingletonTest.java**

~~~~
package com.hundredwordsgof.singleton;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNotNull;
import org.junit.Test;

public class SingletonTest {

  @Test
  public void testSingleton() {

    Singleton singleton = Singleton.getInstance();
    assertNotNull(singleton);

    Singleton secondSingleton = Singleton.getInstance();
    assertEquals(singleton, secondSingleton);
  }
}
~~~~


## Iterator

Una estructura de datos es una forma particular de organizar y almacenar datos, de modo que se pueda acceder y modificar de manera
eficiente. Más precisamente, una estructura de datos es una colección de valores de datos, las relaciones entre ellos y las 
funciones u operaciones que se pueden aplicar a los datos.

Existen numerosos tipos de estructuras de datos, como listas vinculadas, arrays, vectores, mapas, etc. cada coleccion de datos tiene su
propia estructura y su propia forma de acceder a los elementos de la coleccion.

En la práctica, no es conveniente acceder a cada tipo de colección de una manera diferente, por lo que sería bueno tener una 
interfaz común para el acceso elemento a elemento a una colección, independientemente de la forma de la colección.

El patrón Iterator te permite hacer todo esto. La idea clave es tomar la responsabilidad del acceso y transversal fuera del objeto 
agregado y colocarlo en un objeto Iterador que define un protocolo transversal estándar.

Por lo tanto, un patrón de iterador proporciona una forma de acceder a los elementos de un objeto agregado de forma secuencial, sin 
exponer su representación subyacente.

![imagen](http://www.design-patterns-stories.com/assets/img/uml/iterator.png)

### Implementcion

**Iterator.java**

~~~~
package com.hundredwordsgof.iterator;

public interface Iterator {

  Object first();

  Object next();

  boolean isDone();

  Object currentItem();
}
~~~~

**ConcreteIterator.java**

~~~~
package com.hundredwordsgof.iterator;

public class ConcreteIterator implements Iterator {

  private ConcreteAggregate concreteAggregate;
  private int index = -1;

  public ConcreteIterator(ConcreteAggregate concreteAggregate) {
    this.concreteAggregate = concreteAggregate;
  }

  public Object first() {
    index = 0;
    return concreteAggregate.getRecords()[index];
  }

  public Object next() {
    index++;
    return concreteAggregate.getRecords()[index];
  }

  public boolean isDone() {

    if (concreteAggregate.getRecords().length == (index + 1)) {
      return true;
    }
    return false;
  }

  public Object currentItem() {
    return concreteAggregate.getRecords()[index];
  }
}
~~~~

**Aggregate.java**

~~~~
package com.hundredwordsgof.iterator;

public interface Aggregate {

  Iterator createIterator();
}
~~~~

**ConcreteAggregate.java**

~~~~
package com.hundredwordsgof.iterator;

public class ConcreteAggregate implements Aggregate {

  private final String records[] = { "first", "second", "third", "fourth" };

  public Iterator createIterator() {
    return new ConcreteIterator(this);
  }

  protected String[] getRecords() {
    return records;
  }
}
~~~~

### uso

**IteratorTest.java**

~~~~
package com.hundredwordsgof.iterator;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class IteratorTest {

  @Test
  public void testIterator() throws Exception {

    Aggregate aggregate = new ConcreteAggregate();

    Iterator iterator = aggregate.createIterator();

    assertEquals("first", iterator.first());
    assertEquals("first", iterator.currentItem());
    assertEquals(false, iterator.isDone());

    assertEquals("second", iterator.next());
    assertEquals("second", iterator.currentItem());
    assertEquals(false, iterator.isDone());

    assertEquals("third", iterator.next());
    assertEquals("third", iterator.currentItem());
    assertEquals(false, iterator.isDone());

    assertEquals("fourth", iterator.next());
    assertEquals("fourth", iterator.currentItem());
    assertEquals(true, iterator.isDone());
  }
}
~~~~

## Adapter

Imagina que necesitamos desarrollar un editor gráfico que debería poder dibujar varias formas gráficas como líneas, círculos,
rectángulos y texto. Todos nuestros elementos gráficos son subclase de la clase base Forma. Entonces, tendremos LineShape, CircleShape,
RectangeShape y TextShape.

La implementación de TextShape no es fácil. Necesitamos implementar muchas funcionalidades complejas, como el almacenamiento de búfer
de texto, texto en negrita, colorear texto, deshacer, rehacer, "lo que ves es lo que obtienes", etc. Hemos encontrado una biblioteca 
de texto de código abierto que implementa prácticamente todo de la funcionalidad textual que buscamos.

¿Por qué no adaptar una biblioteca de texto existente, para que podamos reutilizar la funcionalidad ya implementada para nuestro 
editor gráfico? Pero, para utilizar la biblioteca de texto existente, debemos adaptar las interfaces de la biblioteca de texto 
existente a nuestras interfaces.

El proceso de adaptación de las interfaces existentes es un ejemplo del patrón del adaptador.

El adaptador nos permite acceder a la interfaz de una clase existente desde otra interfaz.


![Class Adapter](http://www.design-patterns-stories.com/assets/img/uml/classadapter.png)
![Object Adapter](http://www.design-patterns-stories.com/assets/img/uml/objectadapter.png)

### implementacion

**Target.java**
~~~~
package com.hundredwordsgof.adapter.clazz;

public interface Target {
	String request();	
}
~~~~
**Adaptee.java**
~~~~
package com.hundredwordsgof.adapter.clazz;

public class Adaptee {

	public String specialRequest(){
		return "specialRequest";
	}
}
~~~~
**Adapter.java**
~~~~
package com.hundredwordsgof.adapter.clazz;

public class Adapter extends Adaptee implements Target {

	public String request() {
		return this.specialRequest();
	}
}
~~~~
**Target.java**
~~~~
package com.hundredwordsgof.adapter.object;

public interface Target {
	String request();	
}
~~~~
**Adaptee.java**
~~~~
package com.hundredwordsgof.adapter.object;

public class Adaptee {
	public String specialRequest(){
		return "specialRequest";
	}
}
~~~~

**Adapter.java**

~~~~
package com.hundredwordsgof.adapter.object;

public class Adapter implements Target {

  private Adaptee adaptee;

  public Adapter(Adaptee adaptee) {
    this.adaptee = adaptee;
  }

  public String request() {
    return adaptee.specialRequest();
  }
}
~~~~
### uso

**ClazzAdapterTest.java**
~~~~
package com.hundredwordsgof.adapter;

import static org.junit.Assert.*;
import org.junit.Test;
import com.hundredwordsgof.adapter.clazz.Adapter;
import com.hundredwordsgof.adapter.clazz.Target;

public class ClazzAdapterTest {
  @Test
  public void testAdapter() {
    Target target = new Adapter();
    assertEquals("specialRequest", target.request());
  }
}
~~~~
**ObjectAdapterTest.java**
~~~~
package com.hundredwordsgof.adapter;

import static org.junit.Assert.assertEquals;
import org.junit.Test;
import com.hundredwordsgof.adapter.object.Adaptee;
import com.hundredwordsgof.adapter.object.Adapter;
import com.hundredwordsgof.adapter.object.Target;

public class ObjectAdapterTest {

  @Test
  public void testAdapter() {
    Adaptee adaptee = new Adaptee();
    Target target = new Adapter(adaptee);
    assertEquals("specialRequest", target.request());
  }
}
~~~~

## Factory Method

Imagina que necesitamos desarrollar una biblioteca de informes. Dos abstracciones básicas en esta biblioteca son las clases Engine 
y Report. Ambas clases son abstractas, y los clientes tienen que extenderlas para poder realizar implementaciones específicas de 
sus aplicaciones.

La clase de motor es responsable de administrar los informes y los creará según sea necesario. Las subclases de informes que 
Engine debe instanciar son específicas de la aplicación y Engine solo sabe cuándo se debe crear un nuevo informe, pero no qué 
tipo de informe crear. Esto nos lleva a una situación en la que nuestra biblioteca debería instanciar clases, pero solo conoce las
clases abstractas, que no puede instanciar.

Entonces, ¿cómo podemos resolver esto?

Si encapsulamos el conocimiento de las subclases del Informe para crear y mover este conocimiento fuera de la biblioteca, 
entonces la subclase Engine podrá crear objetos del Informe. Esta solución es el factory Method.

El factory Method define una interfaz para crear objetos, pero permite a las subclases decidir qué clase crear una instancia.

![imagen](http://www.design-patterns-stories.com/assets/img/uml/factorymethod.png)

### implementacion

**Product.java**
~~~~
package com.hundredwordsgof.factorymethod;

public interface Product {
}
~~~~
**ConcreteProductA.java**
~~~~
package com.hundredwordsgof.factorymethod;

public class ConcreteProductA implements Product {
}
~~~~
**ConcreteProductB.java**
~~~~
package com.hundredwordsgof.factorymethod;

public class ConcreteProductB implements Product {
}
~~~~
**Creator.java**
~~~~
package com.hundredwordsgof.factorymethod;

abstract class Creator {
  abstract Product factoryMethod(String type);
}
~~~~
**ConcreteCreator.java**
~~~~
package com.hundredwordsgof.factorymethod;

public class ConcreteCreator extends Creator {

  public Product factoryMethod(String type) {
    if (type.equals("A")) {
      return new ConcreteProductA();
    } else if (type.equals("B")) {
      return new ConcreteProductB();
    }
    return null;
  }
}
~~~~
### uso

**FactoryMethodTest.java**
~~~~
package com.hundredwordsgof.factorymethod;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class FactoryMethodTest {
  @Test
  public void testFactoryMethod() {

    Creator factory = new ConcreteCreator();
    Product productA = factory.factoryMethod("A");
    Product productB = factory.factoryMethod("B");

    assertEquals("com.hundredwordsgof.factorymethod.ConcreteProductA",
        productA.getClass().getName());
    assertEquals("com.hundredwordsgof.factorymethod.ConcreteProductB",
        productB.getClass().getName());
        
    assertEquals(null, factory.factoryMethod(""));
  }
}
~~~~

## Observer

Imagina que estamos desarrollando un juego. Una característica de nuestro juego será el sistema de premios. Los jugadores obtendrán
docenas de distintivos diferentes para completar hitos específicos durante el juego.

Cuando los jugadores pasan o alcanzan un cierto punto en el juego, por ejemplo, saltan por encima de una valla compleja, debemos
capturar esa parte del código y calcular el premio.Pero, ¿cómo deberíamos implementar tal característica?

Un enfoque sería encontrar un lugar en el código donde se completen hitos específicos y extender esos lugares con el código que
calcula los premios. Este enfoque no es flexible, no es intuitivo y hace que nuestro código sea complejo y difícil, y viola el 
principio de responsabilidad única.

Otro enfoque sería crear eventos de premio en el código, donde se completan hitos específicos. 
Los eventos de premio se publican como notificaciones, independientemente de quién reciba la notificación. El sistema de premios 
está escuchando todos los eventos de premios e implementa toda la lógica necesaria.

Esta solución es el patrón Observer. El patrón Observer define una dependencia de uno a muchos entre objetos, de modo que cuando
un objeto cambia de estado, todos sus dependientes son notificados y actualizados automáticamente.

![imagen](http://www.design-patterns-stories.com/assets/img/uml/observer.png)

### implementacion

**Observer.java**
~~~~
package com.hundredwordsgof.observer;

public interface Observer {
  void update();
}
~~~~
**ConcreteObserver.java**
~~~~
package com.hundredwordsgof.observer;

public class ConcreteObserver implements Observer {

  private int observerState;
  private ConcreteSubject subject;
  
  public ConcreteObserver(ConcreteSubject subject) {
    this.subject = subject;
  }

  public void update() {
    observerState = subject.getState();
  }

  protected int getObserverState() {
    return observerState;
  }
}
~~~~
**Subject.java**
~~~~
package com.hundredwordsgof.observer;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

abstract class Subject {

  private List<Observer> observers = new ArrayList<Observer>();

  public void attach(Observer observer) {
    observers.add(observer);
  }

  public void dettach(Observer observer) {
    observers.remove(observer);
  }

  public void notifyObervers() {
    for (Iterator iterator = observers.iterator(); iterator.hasNext();) {
      Observer observer = (Observer) iterator.next();
      observer.update();
    }
  }
}
~~~~
**ConcreteSubject.java**
~~~~
package com.hundredwordsgof.observer;

public class ConcreteSubject extends Subject {

  private int state;

  public int getState() {
    return state;
  }

  public void setState(int state) {
    this.state = state;
    this.notifyObervers();
  }
}
~~~~
### uso

**ObserverTest.java**
~~~~
package com.hundredwordsgof.observer;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class ObserverTest {

  @Test
  public void testObserver() throws CloneNotSupportedException {

    ConcreteSubject subject = new ConcreteSubject();

    Observer observer = new ConcreteObserver(subject);
    subject.attach(observer);
    subject.setState(1);
    
    assertEquals(1, ((ConcreteObserver) observer).getObserverState());
    
    subject.dettach(observer);
    subject.setState(0);

    assertEquals(1, ((ConcreteObserver) observer).getObserverState());
  }
}

## Referencias bibliográficas

https://es.wikipedia.org/wiki/Patr%C3%B3n_de_dise%C3%B1o (wikipedia.org)
http://www.design-patterns-stories.com/patterns (design-patterns-stories.com)
