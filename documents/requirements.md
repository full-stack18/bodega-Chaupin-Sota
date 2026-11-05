# Requirements Document

## Introduction

El sistema **Registro de Ventas de la Bodega «Doña Rosa»** permite al encargado de la bodega registrar productos con su precio y stock, registrar ventas seleccionando productos y cantidades, validar las cantidades solicitadas, descontar el stock al confirmar y calcular automáticamente el descuento del 5 % cuando el subtotal supera S/ 100,00.

El sistema no contempla autenticación de usuarios, reportes, facturación electrónica, ni integración con sistemas externos.

## Glossary

- **Sistema**: La aplicación web de registro de ventas de la bodega «Doña Rosa».
- **Encargado**: La persona que atiende la bodega (Doña Rosa o su dependiente).
- **Producto**: Artículo registrado en el sistema con nombre, precio unitario y stock disponible.
- **Stock**: Cantidad de unidades disponibles de un producto en un momento dado.
- **Venta**: Registro de uno o más productos con sus cantidades que el Encargado confirma para un cliente.
- **Línea_de_Venta**: Elemento de una Venta que asocia un Producto con una cantidad solicitada.
- **Subtotal**: Suma de (precio_unitario × cantidad) para todas las Líneas_de_Venta de una Venta.
- **Descuento**: Reducción económica del 5 % aplicada sobre el Subtotal cuando éste es mayor a S/ 100,00.
- **Total**: Resultado de restar el Descuento al Subtotal.

---

## Requirements

### Requisito 1: Gestión de productos

**Historia de usuario:** Como Encargado, quiero registrar productos con su nombre, precio y stock, para mantener actualizado el catálogo de la bodega.

#### Criterios de aceptación

1. WHEN el Encargado envía el formulario de nuevo producto con nombre (máximo 100 caracteres), precio unitario mayor que S/ 0,00 (hasta S/ 999.999,99 con máximo 2 decimales) y stock entero no negativo, THE Sistema SHALL persistir el producto en la base de datos y mostrar el mensaje «Producto registrado correctamente».
2. WHEN el Encargado solicita la lista de productos, THE Sistema SHALL mostrar todos los productos registrados con su nombre, precio unitario y stock actual.
3. IF el Encargado envía el formulario con nombre vacío, THEN THE Sistema SHALL rechazar el registro y mostrar «El nombre es obligatorio».
4. IF el Encargado envía el formulario con precio menor o igual a S/ 0,00 o con más de 2 decimales, THEN THE Sistema SHALL rechazar el registro y mostrar «El precio debe ser mayor que cero y tener máximo 2 decimales».
5. IF el Encargado envía el formulario con stock negativo o no entero, THEN THE Sistema SHALL rechazar el registro y mostrar «El stock debe ser un número entero no negativo».
6. IF el Encargado envía el formulario con un nombre de producto ya registrado, THEN THE Sistema SHALL rechazar el registro y mostrar «Ya existe un producto con ese nombre».

---

### Requisito 2: Registro de una venta

**Historia de usuario:** Como Encargado, quiero seleccionar productos y cantidades para armar una venta, para atender a un cliente de forma rápida y sin errores.

#### Criterios de aceptación

1. WHEN el Encargado selecciona un producto y especifica una cantidad, THE Sistema SHALL agregar una Línea_de_Venta a la Venta en curso y mostrar el Subtotal actualizado.
2. WHILE la Venta en curso contiene al menos una Línea_de_Venta y aún no ha sido confirmada, THE Sistema SHALL permitir agregar nuevas Líneas_de_Venta y habilitar la opción de confirmar la Venta.
3. WHEN el Encargado elimina una Línea_de_Venta y la Venta en curso conserva al menos una Línea_de_Venta restante, THE Sistema SHALL recalcular y mostrar el Subtotal actualizado.
4. WHEN el Encargado elimina la última Línea_de_Venta de la Venta en curso, THE Sistema SHALL mostrar el Subtotal como S/ 0,00 y deshabilitar la opción de confirmar la Venta.
5. WHEN el Encargado selecciona un Producto que ya tiene una Línea_de_Venta en la Venta en curso, THE Sistema SHALL sumar la nueva cantidad a la cantidad existente en esa Línea_de_Venta y mostrar el Subtotal actualizado.

---

### Requisito 3: Validación de cantidad (R3)

**Historia de usuario:** Como Encargado, quiero que el sistema valide las cantidades ingresadas, para evitar registrar ventas con datos incorrectos.

#### Criterios de aceptación

1. IF el Encargado ingresa una cantidad que no es un entero mayor que cero (incluyendo cero, negativo, decimal, no numérico o vacío), THEN THE Sistema SHALL rechazar la Línea_de_Venta y mostrar el mensaje «La cantidad debe ser un número entero mayor que cero» — **[R3]**.
2. WHEN el Encargado ingresa una cantidad que es un entero mayor que cero, THE Sistema SHALL agregar la Línea_de_Venta a la Venta en curso y mostrar el Subtotal actualizado.

---

### Requisito 4: Validación de stock y actualización (R1 · R4)

**Historia de usuario:** Como Encargado, quiero que el sistema controle el stock disponible al confirmar una venta, para nunca vender más unidades de las que existen.

#### Criterios de aceptación

1. WHEN el Encargado agrega una Línea_de_Venta con cantidad mayor al stock disponible del Producto, THE Sistema SHALL rechazar la Línea_de_Venta y mostrar «Stock insuficiente: disponible [N] unidades» — **[R1]**.
2. WHEN el Encargado agrega una Línea_de_Venta con cantidad igual al stock disponible del Producto, THE Sistema SHALL aceptar la Línea_de_Venta y mostrarla en la Venta en curso — **[R1]**.
3. WHEN el Encargado agrega o actualiza una Línea_de_Venta cuya cantidad acumulada para ese Producto supera el stock disponible, THE Sistema SHALL rechazar el cambio y mostrar «Stock insuficiente: disponible [N] unidades» — **[R1]**.
4. WHEN el Encargado confirma la Venta, THE Sistema SHALL reducir el stock de cada Producto en la cantidad vendida de forma atómica, de modo que todos los stocks se actualicen o ninguno lo haga — **[R4]**.
5. IF la actualización atómica de stock falla, THEN THE Sistema SHALL revertir todos los cambios de stock, mantener la Venta en curso sin cambios y mostrar «Error al confirmar la venta. Intente nuevamente» — **[R4]**.
6. WHEN la Venta es confirmada exitosamente, THE Sistema SHALL mostrar los stocks actualizados de cada Producto vendido.

---

### Requisito 5: Cálculo de descuento y resumen de venta (R2)

**Historia de usuario:** Como Encargado, quiero ver el subtotal, el descuento y el total antes y después de confirmar la venta, para informar correctamente al cliente el monto a pagar.

#### Criterios de aceptación

1. WHEN el Subtotal de la Venta es mayor a S/ 100,00, THE Sistema SHALL calcular el Descuento como el 5 % del Subtotal redondeado a 2 decimales y mostrar Subtotal, Descuento y Total en formato S/ #.##0,00 — **[R2]**.
2. WHEN el Subtotal de la Venta es menor o igual a S/ 100,00, THE Sistema SHALL mostrar el Descuento como S/ 0,00 y el Total igual al Subtotal en formato S/ #.##0,00 — **[R2]**.
3. THE Sistema SHALL recalcular Subtotal, Descuento y Total cada vez que el Encargado agrega, actualiza o elimina una Línea_de_Venta.
4. WHEN el Encargado confirma la Venta, THE Sistema SHALL mostrar el resumen final con Subtotal, Descuento y Total en formato S/ #.##0,00.
