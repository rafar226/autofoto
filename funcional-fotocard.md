# Especificación funcional — Generador de hojas de photocards

## 1. Objetivo
Aplicación web independiente y sencilla para cargar imágenes, organizarlas automáticamente en hojas A4, descargar un PDF a escala real e imprimir en papel fotográfico para recortar photocards o figuritas. No integrar con proyectos existentes.

## 2. Alcance del MVP
- Una sola pantalla responsive, preferentemente con procesamiento local en el navegador: sin cuentas, servidor, base de datos ni almacenamiento permanente.
- Carga múltiple mediante selector o arrastrar y soltar; aceptar JPG, PNG y WebP. Mostrar miniaturas, nombre, orden y controles para eliminar, reordenar y duplicar una imagen (cantidad de copias).
- Configuración de hoja A4 (210 × 297 mm), orientación vertical u horizontal; márgenes configurables en mm (valor inicial 5 mm), separación entre figuras (inicial 2 mm), líneas de corte opcionales.
- Configuración de tamaño en mm:
  - Modo **uniforme** (predeterminado): ancho y alto exactos para todas las figuras, inicialmente 55 × 85 mm.
  - Modo **variable**: cada imagen tiene ancho y alto propios, restringidos por un ancho y alto mínimos y máximos configurables. Valores iniciales sugeridos: ancho 40–65 mm y alto 60–95 mm. Validar que mínimos <= máximos y dimensiones > 0.
  - Mostrar en vivo capacidad estimada de una página para el modo uniforme y cantidad real de páginas para el conjunto cargado. No prometer un número fijo en modo variable: depende de tamaños y algoritmo de distribución.
- Ajuste de imagen por figura: **cubrir y recortar** (predeterminado) o **contener completa** (puede dejar bordes blancos). Nunca estirar. Permitir ajustar manualmente encuadre/posición del recorte por imagen si resulta viable; si no, incluir como mejora posterior y usar recorte centrado en MVP.
- Distribución automática sin superposición, respetando dimensiones físicas, márgenes y separación. En modo uniforme usar cuadrícula centrada; en variable usar algoritmo determinista de filas o empaquetado sencillo, priorizando legibilidad y consistencia sobre optimización perfecta. Crear tantas páginas como haga falta y conservar el orden de imágenes cuando sea posible.
- Vista previa paginada de cada hoja A4, miniaturas a escala proporcional, contador de figuras y páginas; actualizar inmediatamente ante cambios.
- Botón **Descargar PDF** y botón **Imprimir**. El PDF debe ser A4 con medidas físicas exactas y todas las páginas; la impresión debe usar ese diseño. No depender de capturas de pantalla de baja resolución.

## 3. Reglas de impresión y calidad
- Generar PDF con tamaño de página real 210 × 297 mm (o invertido), sin escalado arbitrario. Usar coordenadas convertidas de mm a puntos PDF (1 mm = 72/25,4 pt).
- Insertar las imágenes con la mayor calidad original razonable, sin aumentar artificialmente su resolución. Mostrar advertencia de baja resolución por figura según píxeles disponibles y tamaño impreso; objetivo orientativo de 300 ppp para papel fotográfico.
- Mantener relación de aspecto; respetar recorte o contención en exportación igual que en vista previa.
- Líneas de corte finas opcionales, preferentemente fuera del área de la imagen cuando exista espacio; no dibujar marcas que invadan figuras. Si no hay espacio suficiente, ofrecer borde fino opcional alrededor de cada figura.
- Incluir texto visible cerca de descargar/imprimir: «Imprimir en A4, escala 100 % / tamaño real; desactivar “Ajustar a página”. Comprobar los márgenes admitidos por la impresora». No asumir impresión sin bordes.
- No comprimir agresivamente las imágenes. Avisar cuando el PDF pueda resultar pesado.

## 4. Interfaz propuesta
1. Encabezado: «Generador de photocards» y breve explicación.
2. Panel «Imágenes»: zona de carga, galería, orden, duplicados/cantidad y eliminación.
3. Panel «Configuración»: orientación, márgenes, separación, modo uniforme/variable, dimensiones en mm, mínimos y máximos, ajuste de imagen y marcas de corte.
4. Resumen en vivo: total de figuras (incluidas copias), capacidad por hoja en modo uniforme, hojas necesarias y avisos.
5. Vista previa: hojas A4 navegables, con imágenes y espacios de corte.
6. Acciones destacadas: «Descargar PDF» e «Imprimir»; deshabilitadas sin imágenes o con configuración inválida.

## 5. Cálculos y validaciones
- Área útil: ancho de hoja menos márgenes izquierdo/derecho y alto menos márgenes superior/inferior.
- Modo uniforme: columnas = floor((ancho útil + separación)/(ancho figura + separación)); filas = floor((alto útil + separación)/(alto figura + separación)); capacidad = columnas × filas. Páginas = ceil(total de copias/capacidad).
- Si capacidad = 0, indicar que la figura no cabe con esa configuración y bloquear exportación.
- En modo variable, validar cada figura contra límites y área útil; mostrar un error identificando la imagen que no cabe. Calcular cantidad real de hojas a partir de la distribución implementada.
- Recalcular al agregar/quitar/reordenar imágenes, cambiar cantidades, orientación, dimensiones, márgenes o separación.
- Validar tipos de archivo y errores de lectura; gestionar múltiples archivos sin bloquear la interfaz. Liberar URL temporales al eliminar imágenes o cerrar la vista.

## 6. Exportación Word (opcional, fuera del MVP)
Priorizar PDF porque conserva mejor el tamaño físico y el diseño. Si se implementa DOCX después, fijar página A4, márgenes y dimensiones de imágenes en unidades físicas; advertir que Word puede alterar la paginación según versión, configuración o impresora. No bloquear la entrega del MVP por esta opción.

## 7. Criterios de aceptación
- Se pueden cargar 20 o más imágenes en una operación y generar varias hojas automáticamente.
- Al seleccionar 55 × 85 mm, las figuras del PDF tienen exactamente esas dimensiones físicas; la hoja mide A4.
- El resumen coincide con la cantidad de figuras y páginas efectivamente exportadas.
- Ninguna figura se superpone, queda fuera del área útil ni aparece deformada.
- Cambiar orientación, márgenes o tamaño actualiza vista previa y paginación.
- Descargar PDF funciona sin iniciar sesión y contiene todas las imágenes y páginas en el orden mostrado.
- La opción Imprimir abre el diálogo de impresión con el diseño preparado; se informa que el usuario debe elegir tamaño real/100 %.
- Se muestran errores claros ante imágenes inválidas, dimensiones imposibles o baja resolución.

## 8. Implementación sugerida para Codex
Crear proyecto web autónomo, sin backend. Elegir stack ligero y mantenible. Separar módulos de carga/estado, motor de distribución, vista previa y exportador PDF; utilizar una única estructura de posiciones y dimensiones en mm como fuente de verdad para vista previa y PDF, evitando discrepancias. Incorporar pruebas unitarias para cálculo de cuadrícula, paginación, márgenes, orientación y casos imposibles. Mantener la interfaz en español. No agregar autenticación, pagos, IA, catálogos ni otras funciones ajenas al alcance.
