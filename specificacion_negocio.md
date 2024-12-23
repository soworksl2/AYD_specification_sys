# Especificacion sistematica flujo de trabajo Grupo A&D. Revision 1

Este documento sirve de documentacion o una manera de sistematizacion
para la facilitacion en la realizacion del software del negocio **Grupo
A&D**.

-   NOTE: Esto es un boceto rapido, para mejorar la comunicacion
    sistematica y rigorosa del sistema, Se necesita colaboracion de
    todos los involucrados para areglar errores, o agregar clausulas
    faltantes.

-   NOTE: Todas las caracteristicas que se dejen fuera por motivos de un
    desarrollo rapido, se debe verificar su dificultad para implementar
    dicha caracteristica en el futuro. Y tomar decisiones en base que
    tan necesario es para tener un sistema funcional y que tan dificil
    seria implementarlo mas adelante.

-   NOTE: Puesto que cualquier informe o reporte es facilmente obtenible
    sin importar la arquitectura del sistema, dependiendo de SQL, no se
    hace mucho enfasis en ellos a menos que sea necesario.

## Modulos

Especificacion de los modulos, o partes del sistema de manera modular
que son necesarios para el negocios.

Aqui se excluyen modulos intuibles como el modulo para manejo de
usuarios con funcionalidades como (cambiar contraseña, recuperacion de
cuenta, etc) y solo se enfoca en especificar los modulos necesarios para
entender la logica de negocio. Tambien se especifica la relacion de los
modulos los unos a los otros; y que permisos tienenlos roles en cada
modulo.

### Modulo Inventario

Solamente los Administradores tienen accesso a este Modulo

-   Se debe registrar nuevos productos en el sistema

-   Se debe poder agregar dichos productos al inventario como una
    entrada y un motivo.
    -   El inventario solo se debe interpretar como Entradas y Salidas.
        Los Motivos son datos que se manejara por el negocio
    -   Hay motivos predeterminados en el sistema como `Venta` o
        `Compra`
    -   La salida del inventario se realiza cuando se materializa la
        `Venta` no cuando se crea la factura. (Leer mas adelante)

-   Se puede Sacar productos del inventario como una salida y un motivo

-   Se debera poder crear Motivos personalizados de Entrada y de Salida

-   El inventario puede tener existencia fisica negativa, esto puede
    reflejar una mala gestion y que se necesitaria recontar las
    existencias. Esto para no parar el flujo de negocios si hay
    discrepancias en el sistema y el almacen real

-   Se debera poder hacer un inventario desde cero, una vez confirmado
    este inventario, se debe presentar la diferencias entre el
    inventario actual y este nuevo inventario, si todo esta bien,
    entonces se puede confirmar y este nuevo inventario se convierte en
    el nuevo actual inventario fisico.

-   Al momento de registrar entradas de productos, se debe especificar a
    que almacen. Puesto que el negocio tiene diferentes almacenes.
    Actualmente la logistica para mover productos entre almacenes, la
    dejaremos fuera para un rapido desarrollo.

-   Se deben de visualizar todos los productos en un dashboard, su
    existencia fisica, su existencia logica, y la cantidad en transito
    de manera global o en diferentes almacenes. (Lease mas adelante)

-   \[Opcional\] se debe de visualizar los productos individuales y su
    informacion

-   \[Opcional\] Los productos se deben de poder filtrar por almacenes.

-   Los roles de administrador deben de ser notificados, cuando la
    existencia logica de un producto es baja o es negativa, es decir que
    falta producto para todas las ventas.

-   El inventario debera ser exportable como csv, o cualquier otro
    formato que se decida

-   NOTE: Para preocuparnos por el tiempo, no se incliran datos extras
    como Tallas, Color, Modelo, Etc. Si hay variantes se distinguira por
    el nombre, si es necesario con un formato especial. *Respuesta a
    Gestion de Inventarios pregunta #5*

-   NOTE: El inventario lleva conteo de la existencia fisica de cada
    producto, la Existencia logica es un calculo de las existencias de
    los productos menos todas las facturas en estado pendiente (O lista
    para entregar)

-   NOTE: Actualmente este modulo se encarga el Administrador, pero
    puede que en un futuro se cree el rol de Administrador de Inventario
    (Esto es practicamente para ahorrar tiempo de desarrollo puesto que
    la frontera entre lso roles actualmente se puede dividir en
    vendedores, mensajeros y administradores)

-   NOTE: Esto es con respecto al inventario en tiempo real, el
    historial como entradas y salidas y todo datos que se puedan
    guardar, depende de los desarrolladores y como se vaya desarrollando
    el sistema. Teniendo en cuenta que tan dificil seria implementarlo
    despues y que tan importante es para tener el sistema funcional.

### Modulo Ventas

El modulo de venta deve de ser estilo POINT OF SALE, Y para lo unico que
debe de servir es para iniciar una factura y el proceso que este
conlleva

Solo los Administradores y los vendedores tienen aceso a este modulo,
los Administradores se pueden comportar como vendedores pero tienen
funcionalidades extras.

-   Se debe poder iniciar una nueva venta y especificar todos los
    productos, cantidad y precios para la venta

-   los precios se deben colocar automaticamente a un precio base
    guardado en la BD

-   este precio se vera reflejado con descuentos especificados en la
    base de datos. Actualmente solo por productos individuales. [Vease
    Tecnico:Producto](###Productos)

-   Los vendedores podran introducir informacion del cliente como
    (Numero de telefono, Nombre, Y principalmente Ubicacion Precisa y
    Sector)

-   Al confirmar la venta se crea una factura internamente con los datos del
    vendedor y va al modulo de facturas

-   \[Opcional\] cuando una venta se confirma y se crea la factura,
    seria bueno redirigir al vendedor dentro del sistema a la factura
    creada automaticamente, esto por que reguarmente los vendedores
    imprimen la factura en pdf para enviarla al cliente.

-   Los vendedores deberan de tener un aviso cuando la existencia logica
    sea baja o no haya existencia logica.

-   La venta se puede realizar incluso si no hay inventario fisico, en
    ese caso los administradores son notificados Vease el Modulo de
    inventario

-   NOTE: En la factura El sector es para asignar las facturas a
    mensajeros que trabajen en dicho sector, esto es opcional ya que si
    se encuentra una manera mejor internamente de deducir por la
    localidad del cliente a que mensajero convendria asignar la factura
    para entregar, mucho mejor.

-   NOTE: Los pedidos son adomicilios y estos tienen un coste de envio,
    Actualmente se trabaja que los envios son productos y tienen un
    precio ejemplo \[50, 100, 250\]. hay tres productos ENVIO 50, ENVIO
    100, ENVIO 250. esto es por que al momento de pagar al vendedor sus
    ganancia se calcula con el total de la factura - los coste de envio.

### Modulo de Facturas

Este modulo solo puede acceder los vendedores, y los administradores

-   Las facturas tienen los siguientes estados:

    -   Cancelada: si se cancelo la factura
    -   Pendiente: Que esta esperando por ser confirmada por un
        mensajero para el envio de su mercancia al cliente
    -   Asignada: Que un vendedor ya a aceptado la factura para
        entregarla
    -   Recogida: Que el vendedor ya confirmo la existencia y recogio la
        mercancia para su entrega
    -   Completa: Que el vendedor ya entrego y recibio el pago de la
        mercancia.

-   Los Administradores pueden ver todas las facturas

-   Los Vendedores pueden ver todas las facturas que ellos mismos allan
    creados

-   Se pueden cancelar las facturas, para asignarle el estado de
    cancelada. Un vendedor solo puede cancelar una factura si tiene el
    estado de pendiente, pero no si tiene el estado de asignada en
    adelante. Los administradores tienen mayor control y pueden cancelar
    cualquier factura.

    -   NOTE: Necesita ser abordado mas a detalle por los dueños del
        negocio, pero en el caso de que la factura sea recogida por el
        vendedor y no se haya podido entregar o cualquier percance,
        debera ser notificado al algun administrador para cancelar la
        factura

-   Se debe imprimir las facturas en PDF, Opcional como JPEG o PNG.

-   Los administradores tienen la facultad de retroceder una factura a
    su estado de pendiente por cualquier motivo, para que se pueda
    reasignar nuevamente y otro mensajero la pueda aceptar.

### Modulo de Mensajeros

Los mensajeros son los unicos con acceso a este modulo.

-   Los mensajeros tienen un sector asignado donde trabajan o envian.

-   Los mensajeros solo pueden tener una factura a la ves.

-   Los mensajeros pueden solicitar factura, el sistema entonces,
    buscara entre las facturas pendientes y le asignara una dependiendo
    del sector, o ninguna si no hay en el sistema.

-   Los mensajeros despues de aceptar la factura no pueden solicitar una
    nueva.

-   Los mensajero pueden ver toda la informacion de la factura e incluso
    imprimirla

-   Los mensajeros pueden cancelar la factura que actualmente tienen si
    todavia tienen e estado de asignada, entonces la factura vuelve a
    pendiente

-   Los mensajeros pueden asignar la factura a recogida. esto es cuando
    estos ya hayan recogido la mercancia. Una vez aqui los mensajeros no
    pueden cancelar la factura, debe de ser hecho por un administrador.

-   Una vez el mensajero haya entregado la mercancia y recivido el pago,
    puede marcar la factura como completada. En este caso se libera del
    mensajero y este puede solicitar una nueva factura y empezar el
    proceso nuevamente.

-   NOTE: si no hay facturas, los mensajeros deberan ir cada cierto
    tiempo entrando a la aplicacion y solicitar una factura para ver si
    el sistema ya puede asignarle alguna. Esto es mayormente un problema
    tecnico, para no depender de Websockets u otros servicios de
    terceros para enviar notificaciones y trabajar solo con HTTP asi
    podemos avanzar mas rapidamente. Si el equipo de desarrollo esta
    acostumbrado o maneja estas caracteristicas se puede cambiar.

### Modulo de ganancias

Este modulo solo es para Administradores y Vendedores

-   Los administradores tienen una opcion ejemplo ComboBox para
    seleccionar a cualquier vendedor y ver sus facturas y ganancias en
    cada fechas

-   Los vendedores tienen lo mismo pero solo se pueden seleccionar a
    ellos mismos es decir si fuese un ComboBox, solo pudieran
    seleccionar su nombre o id.

-   Como se calcula las ganancias de los vendedores:

    -   Cada vendedor tiene un porcentaje asignado
    -   Por cada facturas que ellos hayan generado, si esta ya esta
        completa
    -   Se calcula el porcentaje del valor total de dicha factura, sin
        calcular los envios.
    -   Esto Hasta que un administrador marque al vendedor como pago
    -   Ejemplo: Es el dia 5 y el vendedor A que se acaba de unir esta
        generando facturas y un administrador decidio pagarle el 27:
        Entonces se pagara todas las facturas completas de este vendedor
        del dia 5 al 27. y se empezara a contar nuevamente desde el 27.

-   Como no veo una logica clara para tener un historial de pago, se
    podria organizar como "PAGO 1", "Pago 2"... "Pago Actual" donde pago
    Actual es la ganancia en tiempo real del vendedor actualmente. pero
    esto depende de los desarrolladores o de los administradores del
    negocio.

-   \[Opcional\] se deberia poder imprimir las ganancias y las facturas
    que se estan pagando al vendedor en PDF

-   NOTE: Los mensajeros se le pagara aparte

-   NOTE: si un vendedor genero facturas, pero estas no se marcaron como
    completas antes del pago, estas facturas seguiran contando para el
    siguiente. Ejemplo: Si un vendedor genero facturas del dia 1 al 15
    pero no todas se marcaron como pagos y al vendedor se le pago, esas
    facturas no pagas no se le pagaron es ese pago en cuestion, pero si
    se marcan como completas para el siguiente pagos, si deben contar.

## Roles

Los diferentes roles necesarios dentro del sistema, su descripcion,
permisos, y flujo de trabajo. Aunque ciertas caracteristicas esta
especificados en el tema **Modulos** esto sirve para conocer de forma
general los roles del sistema y relacion con puestos en el negocio.

### Vendedores

Son los que tienen acceso al modulo de vendedores, solamente crean
facturas y se ponen en contacto con el cliente, estos crean publicidad
en redes sociales u otro medio. Su funcion esta limitada a contactarse
con el cliente para hacer una venta y crear la factura.

### Mensajeros

Son los que una vez se crean las facturas, buscan los productos al
almacen y entregan la mercancias a los clientes y aceptan el pago. Su
funcion se limita a aceptar facturas pendientes y entregarlas.

### Administradores

Tiene mas control que los otros, Un Administrador tambien puede hacer lo
mismo que un Vendedor. Estos se encargan del conteo del inventario, u
arreglar otros procesos como cancelar una factura asignada a algun
vendedor, esto para velar por la integridad de los datos y el negocio.

# Especificacion Tecnica

Esta parte no debe ser tomada de manera tan rigorosa por los
desarrolladores, es solo una manera de registrar, documentar, y
consolidar entre las partes involucradas la arquitectura e
implementacion del sistema.

Para no estorbar con el desarrollo cualquier arquitectura ya
implementada, se puede notificar para reflejar los cambios en este
documento. en definitiva, es pare expicar de una manera mas tecnica el
funcionamiento del negocio

## Base de datos, Modelos y tipos

Se usa pseudocodigo simplificado apra representar la idea general. No se
expresan detalles como por ejemplo tablas intermedias en la DB para
representar una relacion muchos a muchos.

### Type: Id

Es un alias a cualquier tipo de dato que se escoja para reflejar un
identificador unico en una tabla dentro de la base de dato.

### Type: ImgHandler

Es un alias para representar cualquier tipo de datos que se escoja para
guardar imagenes, principalmente en la base de datos. Un ejemplo podria
ser un String para guardar el link a la imagen o un objeto real para
representar la imagen per se y embeberla dentro de la base de datos. (A
decision de los programadores)

### Producto

-   NOTE: Los triggers pueden ser mejorados, pero lo mantendremos
    sencillo para tener el sistema cuanto antes.
-   NOTE: Faltarian mas datos en productos como (barcode, categorias,
    fechas) pero lo mantendremos simple. (Si ya se a implementado algo
    de codigo, notificar para reflejar aqui).

``` cpp
class Producto{
    Id id;
    string name; //varchar(70)
    ImgHandler? img;
    Decimal price;
    List<ProductPriceTrigger> offerts;
}
```

-   name: nombre del producto
-   img: La imagen principal del producto
-   price: el precio base del producto, es el que aparecera
    automaticamente en las facturas si no se cambia.
-   offers: Un conjunto de ofertas para este producto, Si mas de una
    oferta son activadas al mismo tiempo en una factura, se escojera la
    ultima.

``` cpp
class ProductPriceTrigger{
    Id id;
    PriceTriggerType triggerType;
    PriceOffertType priceType;
    Decimal trigger;
    Decimal offert;
}
```

-   triggerType: El desencadenante del trigger
-   priceType: el tipo de valor de `offert`
-   trigger: el valor del trigger por ejemplo si triggerType es
    `TOTAL_INVOCE` y trigger es `5,000` esta oferta se activara cuando
    el total de la factura sea mayor o igual a `5,000`
-   offert: el monto para la oferta, si es `DISCOUNT` se aplicara como
    un descuento al precio base, si es `NEW_PRICE` se asignara este
    nuevo valor

``` cpp
enum PriceTriggerType{
    TOTAL_INVOICE,
    TOTAL_PRODUCT,
    TOTAL_QUANTITY_PRODUCT
}
```

Las ofertas y los triggers deben ser calculadas teniendo en cuenta los
valores bases

-   `TOTAL_INVOICE`: el trigger depende del total de la factura
-   `TOTAL_PRODUCT`: El trigger depende del total que hace la suma de
    este producto en la factura
-   `TOTAL_QUANTITY_PRODUCT`: el trigger depende de la cantidad totalde
    este producto en la factura.

``` cpp
enum PriceOffertType{
    DISCOUNT,
    NEW_PRICE
}
```

NOTE: Las ofertas solo deben aplicarse en las facturas en los productos
que no se han modificado su precio base.

-   `DISCOUNT`: la oferta se interpreta como un descuento
-   `NEW_PRICE`: la oferta se interpreta como un nuevo precio sobre el
    producto base

### Inventory

NOTE: Vamos a delegar algunas informaciones para despues, para
desarrollar el sistema rapidamente. Por ejemplo: (Facturas de compras)

NOTE: Los inventarios se dividen por almacenes

TODO
