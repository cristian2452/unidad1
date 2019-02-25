# principios de programación solid

## Tabla de contenidos:
- Introducción
- Principio de Responsabilidad Única
- Principio Open/Closed
- Principio de Sustitución de Liskov
- Principio de Segregación de Interfaces
- Principio de Inversión de Dependencias
- Referencias bibliográficas


## Introducción

En ingeniería de software, SOLID (Single responsibility, Open-closed, Liskov substitution, Interface segregation and Dependency 
inversion) es un acrónimo mnemónico introducido por Robert C. Martin a comienzos de la década del 2000 que representa cinco principios
básicos de la programación orientada a objetos y el diseño. Cuando estos principios se aplican en conjunto es más probable que 
un desarrollador cree un sistema que sea fácil de mantener y ampliar con el tiempo. 

Los principios SOLID son guías que pueden ser aplicadas en el 
desarrollo de software para eliminar código sucio provocando que el programador tenga que refactorizar el código fuente hasta que sea 
legible y extensible. Debe ser utilizado con el desarrollo guiado por pruebas o TDD, y forma parte de la estrategia global del 
desarrollo ágil de software y desarrollo adaptativo de software

## Principio de Responsabilidad Única

El principio de Responsabilidad Única nos viene a decir que un objeto debe realizar una única cosa. Es muy habitual, si no prestamos 
atención a esto, que acabemos teniendo clases que tienen varias responsabilidades lógicas a la vez.

**Ejemplo** 

Un ejemplo típico es el de un objeto que necesita ser renderizado de alguna forma, por ejemplo imprimiéndose por pantalla. 
Podríamos tener una clase como esta:

~~~~
	public class Vehicle {
 
    public int getWheelCount() {
        return 4;
    }
 
    public int getMaxSpeed() {
        return 200;
    }
 
    @Override public String toString() {
        return "wheelCount=" + getWheelCount() + ", maxSpeed=" + getMaxSpeed();
    }
 
    public void print() {
        System.out.println(toString());
    }
}
~~~~

Aunque a primera vista puede parecer una clase de lo más razonable, en seguida podemos detectar que estamos mezclando dos 
conceptos muy diferentes: la lógica de negocio y la lógica de presentación.

Hay casos como este que se ven muy claros, pero muchas veces los detalles serán más sutiles y probablemente 
no los detectarás a la primera.

Un solución muy simple sería crear una clase que se encargue de imprimir:

~~~~
public class VehiclePrinter {
    public void print(Vehicle vehicle){
        System.out.println(vehicle.toString());
    }
}
~~~~

## Principio Open/Closed
Este principio nos dice que una entidad de software debería estar abierta a extensión pero cerrada a modificación. ¿Qué quiere 
decir esto? Que tenemos que ser capaces de extender el comportamiento de nuestras clases sin necesidad de modificar su código. 
Esto nos ayuda a seguir añadiendo funcionalidad con la seguridad de que no afectará al código existente. Nuevas funcionalidades 
implicarán añadir nuevas clases y métodos, pero en general no debería suponer modificar lo que ya ha sido escrito.

El principio Open/Closed se suele resolver utilizando polimorfismo. En vez de obligar a la clase principal a saber cómo 
realizar una operación, delega esta a los objetos que utiliza, de tal forma que no necesita saber explícitamente cómo llevarla 
a cabo. Estos objetos tendrán una interfaz común que implementarán de forma específica según sus requerimientos.

**Ejemplo**

Siguiendo con nuestro ejemplo de vehículos, podríamos tener la necesidad de dibujarlos en pantalla. 
Imaginemos que tenemos una clase con un método que se encarga de dibujar un vehículo por pantalla. Por supuesto, 
cada vehículo tiene su propia forma de ser pintado. Nuestro vehículo tiene la siguiente forma:

~~~~
public class Vehicle {
 
    public VehicleType getType(){
        ...
    }
    ...
}
~~~~

Básicamente es una clase que especifica su tipo mediante un enumerado. Podemos tener por ejemplo un enum con un par de tipos:

~~~~
public enum VehicleType {
    CAR,
    MOTORBIKE
}
~~~~

Y éste es el método de la clase que se encarga de pintarlos:

~~~~
public void draw(Vehicle vehicle) {
    switch (vehicle.getType()) {
        case CAR:
            drawCar(vehicle);
            break;
        case MOTORBIKE:
            drawMotorbike(vehicle);
            break;
    }
}
~~~~

Mientras no necesitemos dibujar más tipos de vehículos ni veamos que este switch se repite en varias partes de nuestro código.

Pero puede llegar un punto en el que necesitemos dibujar un nuevo tipo de vehículo, y luego otro… Esto implica crear un nuevo 
enumerado, un nuevo case y un nuevo método para implementar el dibujado. En este caso sería buena idea aplicar el principio 
Open/Closed.

Si lo solucionamos mediante herencia y polimorfismo, el paso evidente es sustituir ese enumerado por clases reales, y que cada 
clase sepa cómo pintarse:

~~~~
public abstract class Vehicle {
 
    ...
 
    public abstract void draw();
}
 
public class Car extends Vehicle {
 
    @Override public void draw() {
        // Draw the car
    }
}
 
public class Motorbike extends Vehicle {
 
    @Override public void draw() {
        // Draw the motorbike
    }
}
~~~~

Ahora nuestro método anterior se reduce a:

~~~~
public void draw(Vehicle vehicle) {
    vehicle.draw();
}
~~~~

Añadir nuevos vehículos ahora es tan sencillo como crear la clase correspondiente que extienda de Vehicle:

~~~~
public class Truck extends Vehicle {
 
    @Override public void draw() {
        // Draw the truck
    }
}
~~~~

Esta clase está guardando la información del objeto y la forma de pintarlo. ¿Implica eso que es incorrecto? No necesariamente, 
tendremos que ver si el hecho de tener el método draw en nuestros objetos afecta negativamente la mantenibilidad y testabilidad 
del código. En ese caso habría que buscar alternativas.

## Principio de Sustitución de Liskov

El principio de sustitución de Liskov nos dice que si en alguna parte de nuestro código estamos usando una clase, y esta clase 
es extendida, tenemos que poder utilizar cualquiera de las clases hijas y que el programa siga siendo válido. Esto nos obliga a 
asegurarnos de que cuando extendemos una clase no estamos alterando el comportamiento de la padre.

**Ejemplo**

En la vida real tenemos claro que un cuadrado es un rectángulo con los dos lados iguales. Si intentamos modelar un cuadrado como una 
concreción de un rectángulo, vamos a tener problemas con este principio:

~~~~
public class Rectangle {
     
    private int width;
    private int height;
 
    public int getWidth() {
        return width;
    }
 
    public void setWidth(int width) {
        this.width = width;
    }
 
    public int getHeight() {
        return height;
    }
 
    public void setHeight(int height) {
        this.height = height;
    }
 
    public int calculateArea() {
        return width * height;
    }
}
~~~~

Y un test que comprueba el área:

~~~~
@Test
public void testArea() {
    Rectangle r = new Rectangle();
    r.setWidth(5);
    r.setHeight(4);
    assertEquals(20, r.calculateArea());
}
~~~~

La definición del cuadrado sería la siguiente:

~~~~
public class Square extends Rectangle {
 
    @Override public void setWidth(int width) {
        super.setWidth(width);
        super.setHeight(width);
    }
 
    @Override public void setHeight(int height) {
        super.setHeight(height);
        super.setWidth(height);
    }
}
~~~~

Prueba ahora en el test a cambiar el rectángulo por un cuadrado. ¿Qué ocurrirá? Este test no se cumple, el resultado sería 16 
en lugar de 20. Estamos por tanto violando el principio de sustitución de Liskov.

¿Cómo lo solucionamos?
Hay varias posibilidades en función del caso en el que nos encontremos. 
Lo más habitual será ampliar esa jerarquía de clases. Podemos extraer a otra clase padre las características comunes y hacer que 
la antigua clase padre y su hija hereden de ella. Al final lo más probable es que la clase tenga tan poco código que acabes teniendo
un simple interfaz. Esto no supone ningún problema en absoluto:

~~~~
public interface IRectangle {
    int getWidth();
    int getHeight();
    int calculateArea();
}
 
public class Rectangle extends IRectangle {
    ...
}
 
public class Square extends IRectangle {
    ...
}
~~~~

Pero para este caso en particular, nos encontramos con una solución mucho más sencilla. La razón por la que no se cumple que un 
cuadrado sea un rectángulo, es porque estamos dando la opción de modificar el ancho y alto después de la creación del objeto. 
Podemos solventar esta situación simplemente usando inmutabilidad.

La inmutabilidad es un tema muy interesante que trataré en algún artículo más adelante. Consiste en que una vez que se ha creado un 
objeto, el estado del mismo no puede volver a modificarse. La inmutabilidad tiene múltiples ventajas, entre ellas un mejor uso de 
memoria (todo su estado es final) o seguridad en múltiples hilos de ejecución. Pero ciñéndonos al ejemplo, ¿cómo nos ayuda aquí la 
inmutabilidad? De esta forma:

~~~~
public class Rectangle {
 
    public final int width;
    public final int height;
 
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
}
 
public class Square extends Rectangle {
 
    public Square(int side) {
        super(side, side);
    }
}
~~~~

Desde el momento de la instanciación del objeto, todo lo que hagamos con él será válido, ya usemos un rectángulo o un cuadrado. 

## Principio de segregación de interfaces

El principio de segregación de interfaces viene a decir que ninguna clase debería depender de métodos que no usa. Por tanto, 
cuando creemos interfaces que definan comportamientos, es importante estar seguros de que todas las clases que implementen esas 
interfaces vayan a necesitar y ser capaces de agregar comportamientos a todos los métodos. En caso contrario, es mejor tener varias 
interfaces más pequeñas.

**Ejemplo**

Imagina que tienes una tienda de CDs de música, y que tienes modelados tus productos de esta manera:

~~~~
public interface Product
{
  String getName();
  int getStock();
  int getNumberOfDisks();
  Date getReleaseDate();
}
 
public class CD implements Product {
  ...
}
~~~~

El producto tiene una serie de propiedades que nuestra clase CD sobrescribirá de algún modo. Pero ahora has decidido ampliar mercado, 
y empezar a vender DVDs también. El problema es que para los DVDs necesitas almacenar también la clasificación por edades, porque 
tienes que asegurarte de que no vendes películas no adecuadas según la edad del cliente. Lo más directo sería simplemente añadir la 
nueva propiedad a la interfaz:

~~~~
public interface Product
{
  ...
  int getRecommendedAge();
}
~~~~

¿Qué ocurre ahora con los CDs? Que se ven obligados a implementar getRecommendedAge(), pero no van a saber qué hacer con ello, 
así que lanzarán una excepción:

~~~~
public class CD implements Product {
 
  ...
 
  @Override
  public int getRecommendedAge()
  {
    throw new UnsupportedOperationException();
  }
}
~~~~

Con todos los problemas asociados que hemos visto antes. Además, se forma una dependencia muy fea, en la que cada vez que añadimos 
algo a Product, nos vemos obligados a modificar CD con cosas que no necesita. Podríamos hacer algo tal que así:

~~~~
public interface DVD extends Product {
  int getRecommendedAge();
}
~~~~

Y hacer que nuestras clases extiendan de aquí. Esto solucionaría el problema a corto plazo, pero hay algunas cosas que pueden 
seguir sin funcionar demasiado bien. Por ejemplo, si hay otro producto que necesite categorización por edades, necesitaremos 
repetir parte de esta interfaz. Además, esto no nos permitiría realizar operaciones comunes a productos que tengan esta 
característica. La alternativa es segregar las interfaces, y que cada clase utilice las que necesite. Tendríamos por tanto una 
interfaz nueva:

~~~~
public interface AgeAware {
  int getRecommendedAge();
}
~~~~

Y ahora nuestra clase DVD implementará las dos interfaces:

~~~~
public class CD implements Product {
  ...
}
 
public class DVD implements Product, AgeAware {
  ...
}
~~~~

La ventaja de esta solución es que ahora podemos tener código AgeAware, y todas las clases que implementen esta interfaz podrían 
participar en código común. Imagina que no vendes sólo productos, sino también actividades, que necesitarían una interfaz diferente. 
Estas actividades también podrían implementar la interfaz AgeAware, y podríamos tener código como el siguiente, independientemente 
del tipo de producto o servicio que vendamos:

~~~~
public void checkUserCanBuy(User user, AgeAware ageAware){
  return user.getAge() >= ageAware.getRecommendedAge();
}
~~~~

## Principio de inversión de dependencias

Gracias al principio de inversión de dependencias, podemos hacer que el código que es el núcleo de nuestra aplicación no dependa de 
los detalles de implementación, como pueden ser el framework que utilices, la base de datos, cómo te conectes a tu servidor… Todos 
estos aspectos se especificarán mediante interfaces, y el núcleo no tendrá que conocer cuál es la implementación real para funcionar.

**Ejemplo**

Imaginemos que tenemos una cesta de la compra que lo que hace es almacenar la información y llamar al método de pago para que ejecute
la operación. Nuestro código sería algo así:

~~~~
public class ShoppingBasket {
 
    public void buy(Shopping shopping) {
 
        SqlDatabase db = new SqlDatabase();
        db.save(shopping);
         
        CreditCard creditCard = new CreditCard();
        creditCard.pay(shopping);
    }
}
 
public class SqlDatabase {
    public void save(Shopping shopping){
        // Saves data in SQL database
    }
}
 
public class CreditCard {
    public void pay(Shopping shopping){
        // Performs payment using a credit card
    }
}
~~~~
Aquí estamos incumpliendo todas las reglas que impusimos al principio. Piensa ahora qué pasa si quieres añadir métodos de pago, o 
enviar la información a un servidor en vez de guardarla en una base de datos local. No hay forma de hacer todo esto sin desmontar 
toda la lógica. ¿Cómo lo solucionamos?

Primer paso, dejar de depender de concreciones. Vamos a crear interfaces que definan el comportamiento que debe dar una clase para 
poder funcionar como mecanismo de persistencia o como método de pago:

~~~~
public interface Persistence {
    void save(Shopping shopping);
}
 
public class SqlDatabase implements Persistence {
     
    @Override
    public void save(Shopping shopping){
        // Saves data in SQL database
    }
}
 
public interface PaymentMethod {
    void pay(Shopping shopping);
}
 
public class CreditCard implements PaymentMethod {
     
    @Override
    public void pay(Shopping shopping){
        // Performs payment using a credit card
    }
}
~~~~

¿Ves la diferencia? Ahora ya no dependemos de la implementación particular que decidamos. Pero aún tenemos que seguir instanciándolo
en ShoppingBasket.

Nuestro segundo paso es invertir las dependencias. Vamos a hacer que estos objetos se pasen por constructor:

~~~~
public class ShoppingBasket {
     
    private final Persistence persistence;
    private final PaymentMethod paymentMethod;
 
    public ShoppingBasket(Persistence persistence, PaymentMethod paymentMethod) {
        this.persistence = persistence;
        this.paymentMethod = paymentMethod;
    }
 
    public void buy(Shopping shopping) {
        persistence.save(shopping);
        paymentMethod.pay(shopping);
    }
~~~~
    
 Y si ahora queremos pagar por Paypal y guardarlo en servidor? Definimos las concreciones específicas para este caso, y se las 
pasamos por constructor a la cesta de la compra:
    
~~~~
public class Server implements Persistence {
 
    @Override
    public void save(Shopping shopping) {
        // Saves data in a server
    }
}
 
public class Paypal implements PaymentMethod {
 
    @Override
    public void pay(Shopping shopping) {
        // Performs payment using Paypal account
    }
}
}
~~~~

Ya hemos conseguido nuestro objetivo. Además, si ahora queremos testear ShoppingBasket, podemos crear Test Doubles para las 
dependencias, de forma que nos permita probar la clase de forma aislada.

## Referencias bibliográficas

- https://devexperto.com (Antonio Leiva)
- https://es.wikipedia.org/wiki/SOLID
