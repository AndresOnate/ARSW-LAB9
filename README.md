### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/ddc0ad93-b685-4b5b-bfc4-b4678fee420a)


2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

   ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/176a3861-e7ce-4c08-8f35-0a2da7c34bad)


4. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
5. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

   ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/f26f086a-d397-4b96-95f1-6fd7aae4760f)

    `cd <your_repo>/FibonacciApp`

    `npm install`

7. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

   ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/00eb6924-6ff8-4201-9fa8-b5032129463c)


9. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/c36308d5-5a03-406f-a0a9-e13828323ea4)


7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000:
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/1af298b3-e387-4cd8-b353-4c20f45a8e7d)
    * 1010000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/6af93b63-83f9-4c58-a643-db8fc5cd0670)
    * 1020000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/ef1b6fcf-963a-4613-892c-b81ecc717e09)
    * 1030000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/dadd45ab-9ee2-45c9-ba3d-c8c224fce780)
    * 1040000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/4faa7202-8de2-456e-9857-5489dece9c70)
    * 1050000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/45d813a6-4fa3-4c0a-91b3-89bff9ed4be0)
    * 1060000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/e82af39a-e661-4400-a019-7eac0e5c4bdc)
    * 1070000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/72e7bbec-e1c5-4552-b799-9d4ec4ee897e)
    * 1080000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/3eff48d9-d5c0-4ecf-b95e-58be9092bb2f)
    * 1090000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/be99b56e-06fa-40b0-9918-cd40ba889452)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/c2e56bae-625c-44c3-92cb-db00f921e230)


9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
    * 1000000:
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/11f8ba6e-2495-46ab-90ca-e6893f9df43c)
    * 1010000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/f61f4190-1326-4fc8-aaf8-3766ab9d34aa)
    * 1020000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/a71b341f-a8a3-4baa-b562-4998202d5d22)
    * 1030000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/0c34da47-555c-4009-8ad9-0f4b90bc6467)
    * 1040000
       ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/9cf19c5c-cc1c-4be0-bbb6-8f90c2905a1b)
    * 1050000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/e5fed17d-c395-4831-9bc1-5f9891448d52)
    * 1060000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/d83cda4c-39f8-417e-bbdd-1cdbd56b8746)
    * 1070000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/586a39dc-db06-49f3-b973-1c6b942c2d6b)
    * 1080000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/0e4b3354-699b-4b51-a8fa-e024c768ab3d)
    * 1090000
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/ab9c4d1a-f5fd-42a3-be8e-377697a84bb7)

      Consumo de CPU:
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/02b3ee8c-4213-4958-a8f3-430fb53428dd)

      Ejecución en postman:
      
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/0feac188-e4d8-46d5-ab17-5576da126177)

      Uso de CPU:

      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/d9729cd7-7579-4c95-9512-4ac7935de453)

13. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
    Podemos apreciar con las pruebas hechas a la pagina web que al escalar verticalmente el recurso los tiempos de respuesta no mejoran, pasamos de un tiempo max de 16.5s a 21.4s con las pruebas hechas con    
    postman. Sin embargo, al analizar las graficas de uso de CPU encontramos que se reduce del 70% al 30%, validando que la capacidad del sistema mejorar considerablemente.
15. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
   
   ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/b7b2ecfd-7426-4735-9cb0-b8a77033d095)

   Se crean 7 recursos:
   - Una maquina virtual VERTICAL-SCALABILITY
   - Una dirección IP pública VERTICAL-SCALABILITY-ip
   - Un grupo de seguridad VERTICAL-SCALABILITY-ip
   - Una red virtual VERTICAL-SCALABILITY-vnet
   - Una interfaz de red VERTICAL-SCALABILITY571_z1
   - Un disco VERTICAL-SCALABILITY-disk
   - Claves SSH VERTICAL-SCALABILITY_Key

2. ¿Brevemente describa para qué sirve cada recurso?

   Máquina virtual: Azure Virtual Machines son instancias de servicio de imagen que proporcionan recursos informáticos a petición y escalables a precios basados en el uso. En general, una máquina virtual se comporta como un servidor: es un equipo dentro de un equipo
   que proporciona al usuario la misma experiencia que tendría con el propio sistema operativo host. En general, las máquinas virtuales están en un espacio aislado del resto del sistema, es decir, el software de una máquina virtual no puede interferir ni alterar el       propio servidor subyacente. Cada máquina virtual proporciona su propio hardware virtual, que incluye CPU, memoria, unidades de disco duro, interfaces de red y otros dispositivos.

   Dirección IP pública: Las direcciones IP públicas permiten a los recursos de Internet la comunicación entrante a los recursos de Azure. Permiten que los recursos de Azure se comuniquen con los servicios de Azure orientados al público e Internet.

   Grupo de seguridad: Los grupos de seguridad  permiten configurar la seguridad de red como una extensión natural de la estructura de una aplicación, lo que le permite agrupar máquinas virtuales y directivas de seguridad de red basadas en esos grupos.

   Red virtual: Azure Virtual Network es un servicio que proporciona el bloque de compilación fundamental para su red privada en Azure. Una instancia del servicio (una red virtual) permite que muchos tipos de recursos de Azure se comuniquen de forma segura entre sí,      Internet y redes locales. Estos recursos de Azure incluyen máquinas virtuales (VM). Una red virtual es similar a una red tradicional con la que trabajaría en su propio centro de datos. Pero aporta ventajas adicionales de la infraestructura Azure, como escala,       
   disponibilidad y aislamiento.

   Interfaz de red: Una interfaz de red (NIC) permite que una máquina virtual de Azure se comunique con Internet, Azure y los recursos locales.

   Disco: Azure Managed Disks es un almacenamiento de bloques duradero y de alto rendimiento diseñado para usarse con Azure Virtual Machines y Azure VMware Solution. Azure ofrece cuatro opciones de almacenamiento en disco: Almacenamiento en disco Ultra, SSD Premium,      SSD estándar y HDD estándar. Azure Managed Disks tienen el precio del nivel más cercano según el tamaño específico del disco y se facturan cada hora

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
   El comando npm FibonacciApp.js inicia el servidor haciendo uso del puerto 3000 por defecto, al cerrar la conexión el proceso se cierra y la aplicación deja de funcionar. En Azure, las reglas de puerto de entrada se usan para controlar el tráfico que puede llegar a 
   las máquinas virtuales (VM) u otros recursos en una red virtual. Estas reglas generalmente se definen como parte de un grupo de seguridad de red (NSG), que es una colección de reglas de seguridad que se pueden aplicar a una o más máquinas virtuales u otros recursos.
   Las reglas de puerto de entrada se utilizan normalmente para controlar el acceso a servicios o aplicaciones que se ejecutan en máquinas virtuales, como servidores web o servidores de bases de datos. Por ejemplo, podría crear una regla de entrada para permitir el 
   tráfico en el puerto TCP 80 (HTTP) desde cualquier dirección IP de origen, pero denegar el tráfico en el puerto TCP 22 (SSH) de todas las direcciones IP confiables, excepto unas pocas. S i no se configura esta rega el trafico que llega al puerto 3000 sería    
   descartado.   
4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

   **Tabla de tiempos sin escalamiento vertical**
   
      | Número | Tiempo   |
      |--------|----------|
      | 1000000| 16.26s   |
      | 1010000| 16.44s   |
      | 1020000| 16.87s   |
      | 1030000| 17.28s   |
      | 1040000| 17.71s   |
      | 1050000| 17.99s   |
      | 1060000| 18.42s   |
      | 1070000| 18.79s   |
      | 1080000| 19.10s   |
      | 1090000| 19.65s   |

   **Tabla de tiempos sin escalamiento vertical**

      | Número   | Tiempo  |
      |----------|---------|
      | 1000000  | 20.51s  |
      | 1010000  | 21.61s  |
      | 1020000  | 20.74s  |
      | 1030000  | 23.10s  |
      | 1040000  | 23.60s  |
      | 1050000  | 23.29s  |
      | 1060000  | 24.54s  |
      | 1070000  | 24.72s  |
      | 1080000  | 25.24s  |
      | 1090000  | 24.85s  |
 
5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

 ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/c2e56bae-625c-44c3-92cb-db00f921e230)

 Al analizar el código de la aplicación encontramos que no se hace uso de un algoritmo eficiente para realizar el calculo del enésimo valor de la secuencia de Fibonnaci:

 ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/cd748fd3-0885-4a2d-847c-870c7fb4498b)
 
Lo que implica un alto consumo de CPU en la VM.
 
6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
      
      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/ebaf643e-f675-4417-8fa1-44406d6f0267)

      ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/9bc4643f-801f-4a16-814c-6fc2d30abe3f)

      Al ejecutar las pruebas desde la máquina con características Standard B1ls el tiempo promedio de 15.8 segundos. Además, a pesar de que las pruebas se ejecutan bajo un número específico, los tiempos de  
      respuesta son distintos.

    * Si hubo fallos documentelos y explique.

      No hubo fallos en la ejecución.
      
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
   ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/7cee9692-e546-47b4-87c5-16055dcc8289)
   
      | Tamaño de Instancia | Características                                                                      |
      |---------------------|------------------------------------------------------------------------------------- |
      | B1ls                | - Procesador: 1 núcleo, 0.5 GB de RAM, 4 GiB de almacenamiento temporal,  $3,80/mes  |
      | B2ms                | - Procesador: 2 núcleos, 8 GB de RAM, 16 GiB de almacenamiento temporal,  $60,74/mes |

    Podemos apreciar que el tamaño B2ms cuanta con una capacidad de procesamiento superior.

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
    Como ya se ha analizado antes, mediante las gráficas de rendimiento, encontramos que es una buena solución este escalamiento vertical. Cuando la aplicación reciba múltiples solicitudes necesitará de 
    mejores características, el tamaño B1ls usaba casi por completo su capacidad, mientras al escalar el recurso, el uso de CPU no llegaba al 50%.
    Se debe considerar cambiar el algoritmo por uno más eficiente.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
    
    ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/255ade6e-a175-41f6-8800-e24ca872147c)

    La máquina virtual se reinicia durante el proceso de ajuste del tamaño, lo que corta la conexión con el recurso.

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
   Sí, con el nuevo tamaño, la máquina virtual dispone de más recursos (Núcleos y memoria RAM) para realizar los cálculos. En la gráfica de rendimiento de la CPU podemos apreciar un mejor uso de los recursos, 
   pero los tiempos de respuesta de las solicitudes son más largos con este tamaño, lo que podemos apreciar al realizar las solicitudes vía postman y la página web.

   ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/0feac188-e4d8-46d5-ab17-5576da126177)

   El tiempo de respuesta para las solicitudes con este tamaño es de 21.1s
         
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

    Datos de la petición 1:

    ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/b72e7ee9-4c91-4724-8181-0f66e7ae90e6)

   Datos de la petición 2:

   ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/1988624c-3e37-47ca-9ed7-27db751b0e92)

   Datos de la petición 3:
   
   ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/a70b7f02-c247-4c26-96c0-4e6ee635b4c7)

   Datos de la petición 4:
   
   ![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/4ce8628f-56f4-4265-96ee-4214082ddf63)

   Los tiempos de respuesta no mejoraron

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

Implementación de los recursos:

![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/8795f101-7954-4f0a-ac31-d33577687bd3)


#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/302c67fa-0700-4b11-9a57-f662daab21a2)


5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
```
![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/cf0dfd45-934c-431e-ad6a-282957c1c30b)
```
http://52.155.223.248/fibonacci/1
```
![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/6af5d067-ff1c-4ec5-85ed-dbed6a32e8bb)


2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

Realizamos las pruebas, dadas las limitaciones de la suscripción de Azure para estudiantes, no se pudo crear la tercera máquina virtual:

![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/4b5d5cc4-0be8-4e4f-b406-92d5a100a59c)

VM1:

![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/028bc876-a19f-49ed-b052-a8becb3acfbd)

VM2:
![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/6f6079c2-5e56-4bdf-8bd2-e88c71ba3ade)


3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```
No es posible agregar otra maquina virtual, ejecutamos las peticiones en paralelo:


![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/61cbefea-d676-49b6-92c9-01908534b714)

![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/4bdfb45b-af24-466d-8196-442942a4ffc5)

VM1:

![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/9160d545-8c05-4b0e-920a-d11fa61ba15b)

VM2:

![image](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/155f96e6-357e-4217-a09b-7cbe6a7fe542)

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?

   Estos son los servicios de equilibrio de carga principales actualmente disponibles en Azure:

   **Azure Front Door** es una red de entrega de aplicaciones que proporciona equilibrio de carga global y un servicio de aceleración de sitios para las aplicaciones web. Ofrece funcionalidades de capa 7 para 
   la aplicación, como la descarga SSL, el enrutamiento basado en rutas, la conmutación por error rápida y el almacenamiento en caché para mejorar el rendimiento y la alta disponibilidad de las aplicaciones.
   
   **Traffic Manager** es un equilibrador de carga de tráfico basado en DNS que le permite distribuir el tráfico de forma óptima a servicios de regiones de Azure globales, al tiempo que proporciona una alta    
   disponibilidad y capacidad de respuesta. Dado que Traffic Manager es un servicio de equilibrio de carga basado en DNS, solo equilibra la carga en el nivel del dominio. Por ese motivo, no puede conmutar por 
   error tan rápidamente como con Azure Front Door, debido a los desafíos comunes relacionados con el almacenamiento en caché de DNS y a los sistemas que no respetan los TTL de DNS.
   
   **Application Gateway** proporciona un controlador de entrega de aplicaciones como servicio, que ofrece diversas funcionalidades de equilibrio de carga de capa 7. Úselo para optimizar la productividad de    
   las granjas de servidores web al traspasar la carga de la terminación SSL con mayor actividad de la CPU a la puerta de enlace.
   
   **Load Balancer** proporciona un servicio de equilibrio de carga de capa 4 con latencia muy baja y alto rendimiento (entrante y saliente) para todos los protocolos UDP y TCP. Se diseñó para administrar 
   millones de solicitudes por segundo, a la vez que garantiza que la solución tiene una alta disponibilidad. Load Balancer tiene redundancia de zona, lo que asegura una alta disponibilidad en todas las zonas 
   de disponibilidad. Admite una topología de implementación regional y una topología entre regiones.

  En Azure, el término SKU (Stock Keeping Unit) se refiere a las unidades de mantenimiento de existencias y se utiliza para describir distintas versiones de un recurso o servicio. Para los balanceadores de 
  carga, las SKU determinan la capacidad, el rendimiento y las características. Algunas SKU comunes para los balanceadores de carga son:
   Básico (Basic):
   - Limitado a una sola instancia de máquina virtual.
   - Sin zonas de disponibilidad.
   Estándar (Standard):
   - Admite múltiples instancias de máquinas virtuales.
   - Permite configurar zonas de disponibilidad para mayor disponibilidad.

   La IP pública en el balanceador de carga sirve como punto de entrada para las aplicaciones y permite una distribución equitativa del tráfico entre las instancias del servicio para mejorar la disponibilidad 
   y el rendimiento.

* ¿Cuál es el propósito del *Backend Pool*?
   El Backend Pool agrupa instancias de backend, como máquinas virtuales, para distribuir equitativamente el tráfico entrante, mejorar la alta disponibilidad mediante la redirección de tráfico en caso de 
   fallos, permitir la escalabilidad horizontal ante aumentos de carga, y facilitar la gestión de diferentes versiones o entornos de aplicaciones. Además, ofrece opciones de afinamiento de rendimiento y la 
   capacidad de dirigir tipos específicos de tráfico a distintos pools, lo que lo convierte en un componente esencial para la eficiencia y confiabilidad de servicios en entornos distribuidos y basados en la 
   nube.

* ¿Cuál es el propósito del *Health Probe*?
  Su propósito principal radica en el monitoreo activo, la detección de fallos y la garantía de la disponibilidad de los servicios. Esta sonda envía solicitudes regulares a las instancias de backend, 
  verificando sus respuestas y asegurándose de que estén en un estado saludable. Si una instancia no cumple con los criterios establecidos, el balanceador de carga puede excluir temporalmente esa instancia del 
  tráfico, contribuyendo así a la escalabilidad dinámica y optimizando el rendimiento general del sistema. La Health Probe es esencial para garantizar que el tráfico se dirija únicamente a instancias de 
  backend que puedan responder eficientemente, mejorando así la fiabilidad y capacidad de respuesta de los servicios.
  
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
  Una Load Balancing Rule tiene como objetivo primordial dirigir de manera eficiente y equitativa el tráfico entrante hacia las instancias de backend. Esta regla permite configurar el enrutamiento basado en 
  diversos criterios, como rutas URL o puertos, proporcionando así una distribución uniforme de las solicitudes entre las instancias disponibles.
  Las sesiones persistentes, como IP Affinity o Cookie Affinity, estas son esenciales para mantener la continuidad de una sesión entre un cliente y un servidor específico. Aunque son fundamentales para 
  garantizar una experiencia de usuario coherente, es crucial considerar cómo afectarán la escalabilidad del sistema, ya que pueden limitar la capacidad del balanceador de carga para distribuir equitativamente 
  las solicitudes entre las instancias de backend, especialmente en situaciones de carga desigual.
  La limitación en la capacidad del balanceador de carga se debe al hecho de que las sesiones persistentes, especialmente las sesiones sticky o basadas en afinidad, vinculan un cliente específico a una 
  instancia particular de backend durante la duración de su sesión. Siempre que ese cliente realice solicitudes, el balanceador de carga debe enviarlas a la misma instancia de backend para mantener la 
  persistencia de la sesión.
  
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
  Una Virtual Network (Red Virtual) es una red aislada y definida por software que permite a las máquinas virtuales (VMs) y otros recursos de 
  la nube comunicarse entre sí de manera segura, independientemente de su ubicación geográfica. Proporciona un entorno de red similar al de una red física tradicional, pero con la flexibilidad y la 
  escalabilidad inherentes a la infraestructura en la nube.
  Una Subnet (Subred) es una subdivisión de una red más grande, ya sea física o virtual. En el contexto de las redes virtuales, las subredes se utilizan para segmentar y organizar recursos de manera lógica. 
  Cada subred tiene su propio rango de direcciones IP y puede contener un conjunto específico de recursos de la red.
  Los conceptos de address space (espacio de direcciones) y address range (rango de direcciones) están relacionados con la asignación de direcciones IP dentro de una red virtual y sus subredes:
  Address Space (Espacio de Direcciones): Se refiere al conjunto total de direcciones IP que se pueden usar en una red virtual. Especifica el rango completo de direcciones IP que la red puede abarcar.
  Address Range (Rango de Direcciones): Es un subconjunto del espacio de direcciones y define el rango de direcciones IP que se asigna a una subred específica dentro de la red virtual. Cada subred tiene su    
  propio rango de direcciones único.

* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
  Las Availability Zones (Zonas de Disponibilidad) son centros de datos físicamente separados dentro de una región de servicios en la nube como Azure de Microsoft. Cada zona de disponibilidad es independiente 
  en términos de alimentación, refrigeración y redes, lo que significa que están diseñadas para resistir fallas de manera aislada. La implementación de recursos en diferentes zonas de disponibilidad mejora la 
  resiliencia y la alta disponibilidad de las aplicaciones y servicios.
  Seleccionar tres zonas de disponibilidad es una práctica recomendada para aumentar la redundancia y la resistencia a fallos. Al distribuir recursos en tres zonas, la 
  aplicación o servicio puede mantenerse en funcionamiento incluso si una zona completa experimenta una interrupción. Esto contribuye a la continuidad del servicio y a la capacidad de recuperación frente a 
  fallas en una ubicación específica.
  Cuando una IP es "zone-redundant" (redundante en zonas), significa que la IP está asociada a un recurso que se extiende a través de múltiples zonas de disponibilidad. Por ejemplo, si tienes una máquina 
  virtual (VM) con una dirección IP asociada y esa VM está configurada para ser "zone-redundant", la máquina virtual puede migrar a otra zona de disponibilidad en caso de una falla en la zona actual.

* ¿Cuál es el propósito del *Network Security Group*?
  El propósito del Network Security Group (NSG) en Azure es proporcionar un nivel adicional de seguridad al permitir o denegar el tráfico de red hacia y desde los recursos de Azure, como máquinas virtuales, 
  subredes y redes virtuales. El NSG actúa como un firewall virtual que filtra el tráfico basándose en reglas que especificas.
  
* Informe de newman 1 (Punto 2)
  
   | Métrica                | Escalamiento Vertical       | Escalamiento Horizontal     |
   |------------------------|-----------------------------|-----------------------------|
   | Tiempos de Respuesta   |          21.1 s             |        16.8 s               |
   | Cantidad de Peticiones |  2 peticiones paralelas     |   2 peticiones paralelas    |
   | Costos                 |      $60,74/mes             |           $34.2/mes         |
   | Uso de CPU             |         47,90%              |    93% VM1y 99% VM2         |

  
* Presente el Diagrama de Despliegue de la solución.
  
   ![Deployment Diagram0](https://github.com/AndresOnate/ARSW-LAB9/assets/63562181/8a333837-22d4-49a7-b971-589c81e495f0)



