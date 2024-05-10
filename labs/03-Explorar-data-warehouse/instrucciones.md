# Explora un almacén de datos relacional

## Prerequisitos
Necesitará una [Azure subscription](https://azure.microsoft.com/free) a la que tenga acceso de nivel administrativo.

## Aprovisionar un área de trabajo de Azure Synapse Analytics

*Un área de trabajo* de Azure Synapse Analytics proporciona un punto central para administrar datos y tiempos de ejecución de procesamiento de datos. Puede aprovisionar un área de trabajo mediante la interfaz interactiva en Azure Portal o puede implementar un área de trabajo y recursos dentro de ella mediante un script o una plantilla. En la mayoría de los escenarios de producción, es mejor automatizar el aprovisionamiento con scripts y plantillas para poder incorporar la implementación de recursos en un proceso de desarrollo y operaciones (DevOps) repetible.

En este ejercicio, utilizará una combinación de un script de PowerShell y una plantilla de ARM para aprovisionar Azure Synapse Analytics.

1. Inicie sesión en Azure Portal en https://portal.azure.com.

2. Utilice el botón **[>_]** a la derecha de la barra de búsqueda en la parte superior de la página para crear un nuevo Cloud Shell en Azure Portal, seleccionando un entorno de **PowerShell** y creando almacenamiento si se le solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel en la parte inferior de Azure Portal, como se muestra aquí:

  ![Azure portal with a cloud shell pane](../03-Explorar-data-warehouse/images/cloud-shell.png)

> **Nota** : si anteriormente creó un shell de nube que usa un entorno Bash , use el menú desplegable en la parte superior izquierda del panel de shell de nube para cambiarlo a PowerShell.

3. Tenga en cuenta que puede cambiar el tamaño del shell de la nube arrastrando la barra separadora en la parte superior del panel o usando los íconos — , ◻ y X en la parte superior derecha del panel para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulte la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview)..
   
4. En el panel de PowerShell, ingrese los siguientes comandos para clonar este repositorio:

```
rm -r laboratorio -f
git clone https://github.com/luiscardpuan/Azure-Data.git laboratorio
```

5. Una vez clonado el repositorio, ingrese los siguientes comandos para cambiar a la carpeta de esta práctica de laboratorio y ejecutar el script setup.ps1 que contiene:

  ```
  cd laboratorio/labs/03-Explorar-data-warehouse
  ./setup.ps1
  ```
6. Si se le solicita, elija qué suscripción desea usar (esto solo sucederá si tiene acceso a varias suscripciones de Azure).

7. Cuando se le solicite, ingrese una contraseña adecuada que se establecerá para su grupo de SQL de Azure Synapse.

> **Nota** : ¡Asegúrese de recordar esta contraseña!

Espere a que se complete el script; esto suele tardar unos 15 minutos, pero en algunos casos puede tardar más.

## Explora el esquema del almacén de datos

En esta práctica de laboratorio, el almacén de datos se hospeda en un grupo de SQL dedicado en Azure Synapse Analytics.

Inicie el grupo SQL dedicado
Una vez completado el script, en Azure Portal, vaya al grupo de recursos **rg-lab-xxxxxxx** que creó y seleccione su área de trabajo de Synapse.
En la página Descripción general de su espacio de trabajo de Synapse, en la tarjeta Abrir Synapse Studio , seleccione Abrir para abrir Synapse Studio en una nueva pestaña del navegador; iniciar sesión si se le solicita.
En el lado izquierdo de Synapse Studio, use el ícono **&rsaquo;&rsaquo;** para expandir el menú; esto revela las diferentes páginas dentro de Synapse Studio que se utilizan para administrar recursos y realizar tareas de análisis de datos.
En la página Administrar , asegúrese de que la pestaña Grupos de SQL esté seleccionada y luego seleccione el grupo de SQL dedicado **sql-xxxxxxx** y use su ícono **&#9655;** para iniciarlo; confirmando que desea reanudarlo cuando se le solicite.
Espere a que se reanude el grupo de SQL. Esto puede tardar unos minutos. Utilice el botón **&#8635;** Actualizar para comprobar su estado periódicamente. El estado se mostrará como En línea cuando esté listo.
