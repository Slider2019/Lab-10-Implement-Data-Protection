# Lab 10 - Implement Data Protection

![Fondo laboratorio](./images/FondoREADME.png)

## Indice

- [Descripción del laboratorio](#descripción-del-laboratorio)  
- [Escenario del laboratorio](#escenario-del-laboratorio)  
- [Esquema Visual del Laboratorio](#esquema-visual-del-laboratorio)  
- [Habilidades adquiridas](#habilidades-adquiridas)  
- [Costo Total del Laboratorio](#costo-total-del-laboratorio)  
- [Desarrollo del laboratorio](#desarrollo-del-laboratorio)  
  - [Tarea 1: Usar una plantilla para aprovisionar infraestructura](#tarea-1-usar-una-plantilla-para-aprovisionar-infraestructura)  
  - [Tarea 2: Crear y configurar un Recovery Services vault](#tarea-2-crear-y-configurar-un-recovery-services-vault)  
  - [Tarea 3: Configurar backup a nivel de máquina virtual](#tarea-3-configurar-backup-a-nivel-de-máquina-virtual)  
  - [Tarea 4: Monitorear Azure Backup](#tarea-4-monitorear-azure-backup)  
  - [Tarea 5: Habilitar replicación de máquina virtual](#tarea-5-habilitar-replicación-de-máquina-virtual)  
- [Conceptos reforzados](#conceptos-reforzados)  
- [Resultados esperados](#resultados-esperados)  
- [Limpieza de recursos](#limpieza-de-recursos)  
- [Contribuciones](#contribuciones)  
- [Licencia](#licencia)  

---

## Descripción del laboratorio

En este laboratorio profundizamos en las capacidades de **protección de datos en Azure**, abordando tanto la prevención de pérdidas accidentales como la preparación frente a desastres. A través de la creación de un **Recovery Services vault**, aprendemos a configurar políticas de respaldo que permiten proteger máquinas virtuales con copias periódicas y retención definida. Además, exploramos cómo **Azure Site Recovery** habilita la replicación de cargas de trabajo críticas hacia regiones secundarias, asegurando que la infraestructura pueda mantenerse operativa incluso ante fallos regionales.  
El laboratorio no solo nos guía en la configuración técnica, sino que también nos introduce en buenas prácticas de seguridad, como el uso de **soft delete** para evitar eliminaciones definitivas no intencionadas y la opción de **Cross Region Restore**, que amplía las posibilidades de recuperación en escenarios reales de continuidad de negocio.

---

## Escenario del laboratorio

La organización se enfrenta al desafío de garantizar la disponibilidad y resiliencia de sus servicios en la nube. En este contexto, se busca implementar una estrategia integral que cubra dos aspectos clave:  

1. **Respaldo y restauración de máquinas virtuales** frente a pérdidas de datos ocasionadas por errores humanos, ataques maliciosos o fallos técnicos.  
2. **Recuperación ante desastres** mediante la replicación de máquinas virtuales hacia una región secundaria, lo que permite mantener la operación incluso si la región principal queda inhabilitada.  

---

## Esquema Visual del Laboratorio

![Diagrama Laboratorio](./images/Esquemalab10.png)

- **Tarea 1:** Usar una plantilla para aprovisionar una infraestructura.  
- **Tarea 2:** Crear y configurar un Recovery Services vault.  
- **Tarea 3:** Configurar el respaldo a nivel de máquina virtual de Azure.  
- **Tarea 4:** Monitorear Azure Backup.  
- **Tarea 5:** Habilitar la replicación de máquina virtual.  

---

## Habilidades adquiridas

- Aprovisionamiento de infraestructura mediante plantillas ARM.  
- Creación y configuración de **Recovery Services vaults**.  
- Implementación de políticas de backup y retención en máquinas virtuales.  
- Monitoreo de operaciones de backup con **diagnostic settings** y almacenamiento de logs.  
- Configuración de replicación de máquinas virtuales con **Azure Site Recovery**.  

---

## Costo Total del Laboratorio

### 1. App Service Plan Premium V3 (P1v3 – East US)

- **Precio por instancia:** $0.20 USD/hora  
- **Costo mensual (744 horas):** ~$149 USD  
- **Costo real por 1 hora de uso:** **$0.20 USD**  
- Incluye 2 núcleos, 8 GB RAM y almacenamiento SSD.  

---

### 2. Azure Backup (Recovery Services Vault – East US)

- **Costo por instancia protegida (VM ≤ 500 GB):** $10 USD/mes  
- **Almacenamiento GRS (Geo-redundant):** $0.0448 USD/GB/mes  
- Ejemplo: 100 GB respaldados ≈ $4.48 USD/mes  
- **Costo real por 1 hora de uso:** proporcional ≈ **$0.02 USD**  
- **Total Backup aproximado (1 hora):** **$0.02 USD**  

---

### 3. Azure Load Testing

En el laboratorio se aclara que el costo se cobra por **usuario virtual/hora total de ejecución**, no por cada usuario individual de manera separada.  

- **Precio por Virtual User Hour (VUH):** $0.15 USD  
- **Ejemplo de prueba con 50 usuarios virtuales durante 1 hora:**  
  - 50 VUH × $0.15 = **$7.50 USD**  

---

### 4. Azure Site Recovery (replicación VM East US → West US)

- **Costo por instancia protegida (después de 31 días):** $25 USD/mes  
- **Costo real por 1 hora de uso:** proporcional ≈ **$0.4 USD**  
- Incluye licencia de replicación; se suman costos de almacenamiento y transacciones.

---

## Estimación total por 1 hora de laboratorio

- **App Service Plan Premium V3:** ~$0.20 USD  
- **Azure Backup (con 100 GB):** ~$0.02 USD  
- **Azure Load Testing (1 VUH):** ~$0.15 USD  
- **Azure Site Recovery:** ~$0.03 USD  
- **Total aproximado:** **~$0.4 USD por 1 hora de uso**

> ⚠️ Nota: Este cálculo refleja un escenario de laboratorio con uso puntual de 1 hora.  
> El costo mensual completo (~$196.50 USD) solo aplica si los recursos permanecen activos todo el mes.

Fuentes:

- [Azure App Service on Windows Pricing](https://azure.microsoft.com/en-us/pricing/details/app-service/windows/)
- [Azure Backup Service](https://azure.microsoft.com/en-us/pricing/details/backup/)
- [Azure Load Testing](https://web-v2.prod.trustradius.com/products/azure-load-testing/pricing)
- [Azure Site Recovery pricing](https://azure.microsoft.com/en-us/pricing/details/site-recovery/)

---

## Desarrollo del laboratorio

### Tarea 1: Usar una plantilla para aprovisionar infraestructura

En esta tarea **usaremos una plantilla para desplegar una máquina virtual**. La máquina virtual será utilizada para probar diferentes escenarios de respaldo.

1. **Descargamos** el archivo principal desde la [web oficial](https://microsoftlearning.github.io/AZ-104-MicrosoftAzureAdministrator/) de los labs del AZ-104 y extraemos el contenido de la carpeta `\Allfiles\Lab10\`.
![Pagina descarga](./images/1.png)

2. **Iniciamos sesión** en el [Azure Portal](https://portal.azure.com).
3. **Buscamos y seleccionamos** la opción **Deploy a custom template**.
![Deploy a custom template](./images/2.png)

4. En la página de implementación personalizada, **seleccionamos** **Build your own template in the editor**.
![Construyendo nuestro propio template](./images/3.png)

5. En la página de edición de la plantilla, **hacemos clic en** **Load file**.
![Construyendo nuestro propio template](./images/4.png)

6. **Localizamos y seleccionamos** el archivo `\Allfiles\Lab10\az104-10-vms-edge-template.json` y luego **abrimos** el archivo.
![Construyendo nuestro propio template](./images/5.png)

   > ⚠️ Tomamos un momento para revisar la plantilla: se desplegará una red virtual y una máquina virtual para demostrar escenarios de respaldo y recuperación.

7. **Guardamos** los cambios.  
8. **Seleccionamos** **Edit parameters** y luego **Load file**.
![Construyendo nuestro propio template](./images/6.png)

9. **Cargamos y seleccionamos** el archivo `\Allfiles\Lab10\az104-10-vms-edge-parameters.json`.
10. **Guardamos** los cambios.
![Construyendo nuestro propio template](./images/7.png)

11. **Completamos** los campos de la implementación personalizada con la siguiente información (dejando los demás valores por defecto):  
    - **Subscription:** Nuestra suscripción de Azure  
    - **Resource group:** `az104-rg-region1` (si es necesario, seleccionamos **Create new**)  
    - **Region:** East US  
    - **Username:** `localadmin`  
    - **Password:** Proporcionamos una contraseña compleja  
12. **Seleccionamos** **Review + Create** y luego **Create**.
![Construyendo nuestro propio template](./images/8.png)

>⚠️ **Esperamos** a que la plantilla se despliegue y luego **seleccionamos** **Go to resource**.  Al finalizar, debemos tener **una máquina virtual en una red virtual**.
![Construyendo nuestro propio template](./images/9.png)

### Tarea 2: Crear y configurar un Recovery Services vault

En esta tarea **crearemos un Recovery Services vault**, el cual proporciona almacenamiento para los datos de las máquinas virtuales.

1. En el **Azure Portal**, **buscamos y seleccionamos** **Recovery Services vaults** y, en el panel correspondiente, **hacemos clic en** **+ Create**.
![Recovery Services Vault](./images/10.png)
![Recovery Services Vault](./images/11.png)

2. En la página de creación del Recovery Services vault, **especificamos** los siguientes valores:

   - **Subscription:** el nombre de nuestra suscripción de Azure  
   - **Resource group:** `az104-rg-region1`  
   - **Vault Name:** `az104-rsv-region1`  
   - **Region:** East US 

   ⚠️ Es importante que indiquemos la misma región en la que desplegamos las máquinas virtuales en la tarea anterior.  
![Recovery Services Vault](./images/12.png)

3. **Seleccionamos** **Review + Create**, verificamos que la validación sea correcta y luego **hacemos clic en** **Create**.
   - Esperamos a que la implementación se complete (toma un par de minutos).  
4. Una vez finalizada la implementación, **hacemos clic en** **Go to Resource**.  
5. En la sección **Settings**, **seleccionamos** **Properties**.
![Recovery Services Vault](./images/13.png)

6. Bajo la etiqueta **Backup Configuration**, **hacemos clic en** **Update**.
![Recovery Services Vault](./images/14.png)

   - En la página de configuración de backup, revisamos las opciones de tipo de replicación de almacenamiento.  
   - Dejamos la configuración predeterminada en **Geo-redundant** y cerramos la página.
   ![Recovery Services Vault](./images/15.png)
   ⚠️ Este ajuste solo puede configurarse si no existen elementos de respaldo ya creados.  
   > 💡 ¿Sabías que? La opción **Cross Region Restore** permite restaurar datos en una región secundaria emparejada de Azure.  
7. Bajo la sección **Security Settings > Soft Delete Settings**, **hacemos clic en** **Update**.
![Recovery Services Vault](./images/16.png)
   - En la página de configuración de **Soft Delete**, verificamos que el período de retención sea de **14 días** y cerramos la página.
![Recovery Services Vault](./images/17.png)

   > 💡 ¿Sabías que? Azure ofrece dos tipos de bóvedas: [**Recovery Services vaults**](https://learn.microsoft.com/en-us/azure/backup/backup-azure-recovery-services-vault-overview) y [**Backup vaults**](https://learn.microsoft.com/en-us/azure/backup/backup-vault-overview). La diferencia principal está en las fuentes de datos que pueden respaldarse.

### Tarea 3: Configurar backup a nivel de máquina virtual

En esta tarea **implementaremos el respaldo a nivel de máquina virtual en Azure**. Como parte de este proceso, debemos definir la política de respaldo y retención que aplicará a la máquina virtual. Es importante destacar que diferentes VMs pueden tener asignadas distintas políticas de respaldo y retención.

⚠️ **Nota:** Antes de comenzar esta tarea, verificamos que el despliegue iniciado en la **Tarea 1** se haya completado correctamente.

1. En el panel del **Recovery Services vault**, **hacemos clic en** **Overview** y luego en **+ Backup**.
![Backup VM](./images/18.png)

2. En la página **Backup Goal**, especificamos los siguientes valores:

   - **Where is your workload running?** → Azure  
   - **What do you want to backup?** → Virtual machine  
   - **Seleccionamos** **Backup**.
![Backup VM](./images/19.png)

3. Observamos que existen dos subtipos de políticas: **Enhanced** y **Standard**. Revisamos las opciones y **seleccionamos** **Standard**.
![Backup VM](./images/20.png)

4. En **Backup policy**, **seleccionamos** **Create a new policy**.  
5. Definimos una nueva política de respaldo con los siguientes valores (dejando los demás por defecto):

   - **Policy name:** `az104-backup`  
   - **Frequency:** Daily  
   - **Time:** 12:00 AM  
   - **Timezone:** nuestro huso horario local  
   - **Retain instant recovery snapshot(s) for:** 2 días
![Backup VM](./images/21.png)

6. **Hacemos clic en** **OK** para crear la política y, en la sección **Virtual Machines**, **seleccionamos** **Add**.
![Backup VM](./images/22.png)

7. En la página **Select virtual machines**, **seleccionamos** `az-104-10-vm0`, luego **OK**, y de regreso en la página de **Backup**, **hacemos clic en** **Enable backup**.
![Backup VM](./images/23.png)

   - ⚠️ Esperamos aproximadamente 2 minutos a que se habilite el respaldo.
![Backup VM](./images/24.png)

8. Una vez completado el despliegue, **seleccionamos** **Go to resource**.  
9. En la sección **Protected items**, **hacemos clic en** **Backup items**, y luego en la entrada de **Azure virtual machine**.
![Backup VM](./images/25.png)
![Backup VM](./images/26.png)

10. **Seleccionamos** el enlace **View details** para `az104-10-vm0` y revisamos los valores de **Backup Pre-Check** y **Last Backup Status**.

    - ⚠️ Notamos que el respaldo está en estado **pending**.  
![Backup VM](./images/27.png)

11. **Seleccionamos** **Backup now**, aceptamos el valor predeterminado en el campo **Retain Backup Till**, y luego **hacemos clic en** **OK**.
![Backup VM](./images/28.png)
![Backup VM](./images/29.png)

    - ⚠️ No es necesario esperar a que el respaldo se complete; podemos continuar con la siguiente tarea.  

### Tarea 4: Monitorear Azure Backup

En esta tarea **desplegaremos una cuenta de almacenamiento en Azure** y configuraremos el Recovery Services vault para enviar los registros y métricas hacia dicha cuenta. Este repositorio podrá ser utilizado posteriormente con **Log Analytics** u otras soluciones de monitoreo de terceros.

1. En el **Azure Portal**, **buscamos y seleccionamos** **Storage accounts**.
![Storage para Azure Backup](./images/30.png)

2. En la página de cuentas de almacenamiento, **hacemos clic en** **Create**.
![Storage para Azure Backup](./images/31.png)

3. Definimos la cuenta de almacenamiento con la siguiente información y luego **seleccionamos** **Review + create**:

   - **Subscription:** nuestra suscripción  
   - **Resource group:** `az104-rg-region1`  
   - **Storage account name:** proporcionamos un nombre globalmente único  
   - **Region:** East US  
   - Finalmente, **seleccionamos** **Create**.  
   ⚠️ Esperamos aproximadamente un minuto a que se complete la implementación.
![Storage para Azure Backup](./images/32.png)

4. **Buscamos y seleccionamos** nuestro **Recovery Services vault**.  
5. En el panel **Monitoring**, **seleccionamos** **Diagnostic Settings** y luego **Add diagnostic setting**.
![Storage para Azure Backup](./images/33.png)
![Storage para Azure Backup](./images/34.png)

6. Asignamos el nombre: **Logs and Metrics to storage**.
![Storage para Azure Backup](./images/35.png)

7. Marcamos las siguientes categorías de registros y métricas:  
   - Azure Backup Reporting Data  
   - Addon Azure Backup Job Data  
   - Addon Azure Backup Alert Data  
   - Azure Site Recovery Jobs  
   - Azure Site Recovery Events

8. En **Destination details**, marcamos la opción **Archive to a storage account**.  
9. En el campo desplegable de **Storage account**, **seleccionamos** la cuenta de almacenamiento que desplegamos en esta tarea.
![Storage para Azure Backup](./images/36.png)

10. **Guardamos** la configuración.  
11. Regresamos al **Recovery Services vault**, en el panel **Monitoring** **seleccionamos** **Backup jobs**.
![Storage para Azure Backup](./images/37.png)

12. Localizamos la operación de respaldo de la máquina virtual `az104-10-vm0`.
![Storage para Azure Backup](./images/38.png)

13. **Visualizamos los detalles** (desplazándonos hacia la derecha para encontrar el enlace) del trabajo de respaldo.
![Storage para Azure Backup](./images/39.png)

### Tarea 5: Habilitar replicación de máquina virtual

En esta tarea **habilitaremos la replicación de una máquina virtual** utilizando Azure Site Recovery. Esto nos permitirá contar con una copia de la VM en una región secundaria, asegurando continuidad operativa en caso de fallos regionales.

1. En el **Azure Portal**, **buscamos y seleccionamos** **Recovery Services vaults** y, en el panel correspondiente, **hacemos clic en** **+ Create**.
![Recovery Services Vault](./images/40.png)

2. En la página de creación del Recovery Services vault, **especificamos** los siguientes valores:  
   - **Subscription:** el nombre de nuestra suscripción de Azure  
   - **Resource group:** `az104-rg-region2` (si es necesario, seleccionamos **Create new**)  
   - **Vault Name:** `az104-rsv-region2`  
   - **Region:** West US
    ![Recovery Services Vault](./images/41.png)

   ⚠️ Es importante que indiquemos una región diferente a la de la máquina virtual.  
3. **Seleccionamos** **Review + Create**, verificamos que la validación sea correcta y luego **hacemos clic en** **Create**.
   - Esperamos a que la implementación se complete (toma un par de minutos).  
4. **Buscamos y seleccionamos** la máquina virtual `az104-10-vm0`.  
![Recovery Services Vault](./images/42.png)

5. En el panel **Backup + Disaster recovery**, **seleccionamos** **Disaster recovery**.
![Recovery Services Vault](./images/43.png)

6. En la pestaña **Basics**, verificamos la **Target region**.
![Recovery Services Vault](./images/44.png)

7. **Seleccionamos** **Next: Advanced settings**. Las selecciones de recursos ya estarán configuradas.
![Recovery Services Vault](./images/45.png)

8. Nos desplazamos hacia abajo y **creamos** la cuenta de automatización.  
   ⚠️ Es importante que los valores estén completos, de lo contrario la validación fallará.  
9. **Seleccionamos** **Review + Start replication** y luego **Enable replication**. 
![Recovery Services Vault](./images/46.png)

   - ⚠️ La habilitación de la replicación tomará entre 10 y 15 minutos. Podemos observar las notificaciones en la parte superior derecha del portal mientras se ejecuta.
![Recovery Services Vault](./images/47.png)

10. Una vez completada la replicación, **buscamos y localizamos** el Recovery Services vault `az104-rsv-region2`.
![Recovery Services Vault](./images/48.png)

    - Es posible que tengamos que **refrescar** la página.  
11. En la sección **Protected items**, **seleccionamos** **Replicated items**.
![Recovery Services Vault](./images/49.png)

12. Verificamos que la máquina virtual aparezca como **healthy** en el estado de replicación.
![Recovery Services Vault](./images/50.png)
![Recovery Services Vault](./images/51.png)

    - El estado mostrará la sincronización (iniciando en 0%) y finalmente se marcará como **Protected** una vez completada la sincronización inicial.  
13. **Seleccionamos** la máquina virtual para ver más detalles.  
    - 💡 ¿Sabías que? Es una buena práctica realizar pruebas de **failover** en una VM protegida para validar la recuperación.
![Recovery Services Vault](./images/52.png)

---

## Conceptos reforzados

- **Backup vs. Site Recovery**: diferencias entre protección de datos y recuperación ante desastres.  
- **Recovery Services vault**: almacenamiento seguro para respaldos y réplicas.  
- **Soft delete**: retención de datos eliminados por 14 días como medida de seguridad.  
- **Cross Region Restore**: posibilidad de restaurar datos en regiones secundarias.  

---

## Resultados esperados

- Una máquina virtual protegida con backup diario.  
- Logs y métricas de backup almacenados en una cuenta de almacenamiento.  
- Replicación activa de la VM hacia otra región, con estado de salud validado.  

---

## Limpieza de recursos

En esta tarea **eliminaremos los recursos del laboratorio** para liberar capacidad y minimizar costos en nuestra suscripción de Azure. La forma más sencilla de hacerlo es borrar directamente el grupo de recursos asociado al laboratorio.

⚠️ **Nota importante:** Para eliminar un **Azure Recovery Services vault**, primero debemos quitar todas las dependencias como elementos protegidos, servidores de respaldo y cuentas de almacenamiento. Además, es necesario deshabilitar características de seguridad como **soft delete** antes de poder borrar la bóveda.  
Un ejemplo de [script en PowerShell](https://learn.microsoft.com/es-mx/azure/backup/scripts/delete-recovery-services-vault) está disponible para automatizar este proceso.

Para eliminar los recursos, seguir los pasos al intentar eliminar el Vault. Ademas se puede seguir todo el proceso de manera manual en [el siguiente enlace](https://learn.microsoft.com/es-mx/azure/backup/backup-azure-delete-vault?tabs=portal)


1. En el **Azure Portal**, **seleccionamos** el grupo de recursos, luego **Delete the resource group**.  

2. **Ingresamos** el nombre del grupo de recursos y finalmente **hacemos clic en** **Delete**.  

También podemos realizar la eliminación mediante comandos:

- **Azure PowerShell:**

  ```powershell
  Remove-AzResourceGroup -Name resourceGroupName
  ```

- **Azure CLI:**

  ```bash
  az group delete --name resourceGroupName
  ```

---

## Contribuciones

Este README fue adaptado y enriquecido a partir de los materiales oficiales del laboratorio AZ-104. Se aceptan mejoras en diagramas, ejemplos de costos y traducciones adicionales.

---

## Licencia

Este contenido se basa en los laboratorios oficiales de Microsoft Learning para el examen AZ-104 Microsoft Azure Administrator. Uso educativo y de práctica profesional.
