## Initial System Code
Here is the enhanced pseudocode for the system you will be working with:

```java
class SystemManager {
    processOrder(order) {
        if (order.type == "standard") {
            verifyInventory(order);
            processStandardPayment(order);
        } else if (order.type == "express") {
            verifyInventory(order);
            processExpressPayment(order, "highPriority");
        }
        updateOrderStatus(order, "processed");
        notifyCustomer(order);
    }
 
    verifyInventory(order) {
        // Checks inventory levels
        if (inventory < order.quantity) {
            throw new Error("Out of stock");
        }
    }
 
    processStandardPayment(order) {
        // Handles standard payment processing
        if (paymentService.process(order.amount)) {
            return true;
        } else {
            throw new Error("Payment failed");
        }
    }
 
    processExpressPayment(order, priority) {
        // Handles express payment processing
        if (expressPaymentService.process(order.amount, priority)) {
            return true;
        } else {
            throw new Error("Express payment failed");
        }
    }
 
    updateOrderStatus(order, status) {
        // Updates the order status in the database
        database.updateOrderStatus(order.id, status);
    }
 
    notifyCustomer(order) {
        // Sends an email notification to the customer
        emailService.sendEmail(order.customerEmail, "Your order has been processed.");
    }
}
```

# Solution

- Single Responsibility
    - Se identifica que la clase tiene distintas responsabilidades que no parecen estar en el mismo contexto
        -   Verificar inventario
        -   Procesar pagos
        -   Actualizar ordenes
        -   Notificaciones

- Se identifica que se rompe el principio Open/Close
    - Cada vez que se quiera agregar un nuevo tipo de pago tendremos que modificar el codigo

- Interface Segregation
    - La clase SystemManager no implementa interfaces para las funciones que no le corresponden, ya que las tiene en metodos dentro de ella

# Refactor


``` java

/**
Se abstrae la logica del procesamiento de ordenes a la interfaz PaymentProcess que sera encargada de orquestar el flujo
completo

Esto sera siempre con la logica de cada implementacion

Como en el resto de las demas clases propuestas se agrega el uso de interfaces con procesos como Invetario, PaymentService,
Order y Notificaciones

Con esta abstraccion a PaymentProcess estamos cumpliendo el principio open/close, ya que de requierir un nuevo tipo de 
procesamiento al pago se agregaria una nueva implementacion
*/

interface PaymentProcess{
    void processOrder(order);
}


/**
Implementacion para pagos standar
*/
class PaymentStandarProcess implemenrs PaymentProcess{
    Inventory inventory;
    PaymentService paymentService;
    Order order;
    Notificarion notification;

    @Override
    void processOrder(order){
        inventory.verifyInventory(order);
        processStandardPayment(order);
        order.updateOrderStatus(order, "processed");
        notificarion.notifyCustomer(order);
    }

    private processStandardPayment(order) {
        if (paymentService.process(order.amount)) {
            return true;
        } else {
            throw new Error("Payment failed");
        }
    }
}


/**
Implementacion para pagos express
*/
class PaymentExpressProcess implemenrs PaymentProcess{
    Inventory inventory;
    PaymentService paymentService;
    Order order;
    Notificarion notification;

    @Override
    void processOrder(order){
        inventory.verifyInventory(order);
        processExpressPayment(order, "highPriority");
        order.updateOrderStatus(order, "processed");
        notificarion.notifyCustomer(order);
    }

    processExpressPayment(order, priority) {
        if (expressPaymentService.process(order.amount, priority)) {
            return true;
        } else {
            throw new Error("Express payment failed");
        }
    }
}


/**
Se agregan interfaces para cumplir con la segregacion de interfaces y responsabilidad unica
*/
interface Invetory{
    verifyInventory(order);
}

class InvetoryImpl implements Inventory{
    
    @Override
    verifyInventory(order){
        if (inventory < order.quantity) {
            throw new Error("Out of stock");
        }
    }
}

/**
Se agregan interfaces para cumplir con la segregacion de interfaces, responsabilidad unica, inyeccion de dependencias
*/
interface Order{
    updateOrderStatus(order, status);
}


/**
Se agregan interfaces para cumplir con la segregacion de interfaces, responsabilidad unica, inyeccion de dependencias
*/
class OrderImp implements Order{

    Database database;

    @Override
    updateOrderStatus(order, status) {
        database.updateOrderStatus(order.id, status);
    }
}


/**
Se agregan interfaces para cumplir con la segregacion de interfaces, responsabilidad unica, inyeccion de dependencias
*/
interface Notification{
    notifyCustomer(order);
}

class NotificationImpl implements Notification{
    private EmailService emailService;

    public NotificationImpl(EmailService emailService){
        this.EmailService = emailService;
    }

    notifyCustomer(order) {
        emailService.sendEmail(order.customerEmail, "Your order has been processed.");
    }
}

/*
La calse system manager la volveremos un clase de "contexto", que sera encargada de determinar la logica aplicar
para el proceso de pago.

Se apoyara de una factorya para obtener la estrategia adecuada al tipo de orden.
**/
class SystemManager{
    Map<String, PaymentProcess> factory;

    processOrder(order) {
        PaymentProcess paymentProcess = factory.get(order.type);
        paymentProcess.processOrder(order);
    }
}

```
