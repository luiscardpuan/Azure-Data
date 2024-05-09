# Explora un almacén de datos relacional

## Prerequisitos
Necesitará una [Azure subscription](https://azure.microsoft.com/free) a la que tenga acceso de nivel administrativo.

## Aprovisionar un área de trabajo de Azure Synapse Analytics

*Un área de trabajo* de Azure Synapse Analytics proporciona un punto central para administrar datos y tiempos de ejecución de procesamiento de datos. Puede aprovisionar un área de trabajo mediante la interfaz interactiva en Azure Portal o puede implementar un área de trabajo y recursos dentro de ella mediante un script o una plantilla. En la mayoría de los escenarios de producción, es mejor automatizar el aprovisionamiento con scripts y plantillas para poder incorporar la implementación de recursos en un proceso de desarrollo y operaciones (DevOps) repetible.

En este ejercicio, utilizará una combinación de un script de PowerShell y una plantilla de ARM para aprovisionar Azure Synapse Analytics.

1. Inicie sesión en Azure Portal en https://portal.azure.com.

2. Utilice el botón **[>_]** a la derecha de la barra de búsqueda en la parte superior de la página para crear un nuevo Cloud Shell en Azure Portal, seleccionando un entorno de **PowerShell** y creando almacenamiento si se le solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel en la parte inferior de Azure Portal, como se muestra aquí:

  ![Azure portal with a cloud shell pane](../images/cloud-shell.png)

  
