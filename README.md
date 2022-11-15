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

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`
    
![image](https://user-images.githubusercontent.com/25957863/201812384-53e876d2-ea4b-4cb0-89d9-adf329f04a3f.png)

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).

![image](https://user-images.githubusercontent.com/25957863/201812437-bfb37a40-3e5c-44c5-97f3-c470d41bea09.png)

5. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

![image](https://user-images.githubusercontent.com/25957863/201812470-086b16bc-06d1-4ec5-aa69-23d1f876d14f.png)

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

![image](https://user-images.githubusercontent.com/25957863/201812502-dc4542de-04ca-43b7-93bb-5710bdb233ad.png)

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. 

![image](https://user-images.githubusercontent.com/25957863/201812537-79670728-398c-40da-abe8-6eaddbc4d821.png)

Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![image](https://user-images.githubusercontent.com/25957863/201812554-3d930832-6745-424c-bb14-d0e52c6b2521.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000    

Calculando Fibonacci para los números dados:

![image](https://user-images.githubusercontent.com/25957863/201812651-349926dc-e1c5-42d5-bec1-60e73ec96bfc.png)

![image](https://user-images.githubusercontent.com/25957863/201812620-497ec270-215f-4f65-89ad-6ed4e6b23aa2.png)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![image](https://user-images.githubusercontent.com/25957863/201812976-c6c415c9-1228-4ec1-b31f-86a90bf34dc6.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).

![image](https://user-images.githubusercontent.com/25957863/201815228-68ce4308-c01e-4914-bcd2-f39239fcae5a.png)

Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.

![image](https://user-images.githubusercontent.com/25957863/201815272-2b6026e1-64f4-4866-8363-57f5872749ec.png)

Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

![image](https://user-images.githubusercontent.com/25957863/201815292-185f0439-7d38-4136-8927-7e676de543a3.png)


10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

![image](https://user-images.githubusercontent.com/25957863/201815749-f2153081-be7b-4cfc-9483-5e1a3333fcc0.png)

2. ¿Brevemente describa para qué sirve cada recurso?

- Azure Virtual Network (VNet) es el bloque de creación fundamental de una red privada en Azure. VNet permite muchos tipos de recursos de Azure, como Azure Virtual Machines (máquinas virtuales), para comunicarse de forma segura entre usuarios, con Internet y con las redes locales. VNet es similar a una red tradicional que funcionaría en su propio centro de datos, pero aporta las ventajas adicionales de la infraestructura de Azure, como la escala, la disponibilidad y el aislamiento.

- Una interfaz de red permite que una máquina virtual de Azure se comunique con los recursos de Internet, Azure y locales. Una máquina virtual creada con el Azure Portal tiene una interfaz de red con la configuración predeterminada. En su lugar, puede crear interfaces de red con una configuración personalizada y agregar una o varias interfaces de red a una máquina virtual al crearla. También puede cambiar la configuración predeterminada de la interfaz de red en una interfaz de red existente.

- Las máquinas virtuales de Azure permiten la implementación de una serie de aplicaciones y programas informáticos en un sistema virtualizado. Además, Azure también permite la virtualización de sistemas operativos Linux y de aplicaciones de Oracle, SAP o IBM, entre otros.

- Las direcciones IP públicas permiten a los recursos de Internet la comunicación entrante a los recursos de Azure. Permiten que los recursos de Azure se comuniquen con los servicios de Azure orientados al público e Internet. Hasta que cancele la asignación, la dirección estará dedicada al recurso. Un recurso sin una dirección IP pública asignada puede realizar comunicaciones salientes. Azure asigna dinámicamente una dirección IP disponible que no está dedicada al recurso.

- Puede usar el grupo de seguridad de red de Azure para filtrar el tráfico de red entre los recursos de Azure de una red virtual de Azure. Un grupo de seguridad de red contiene reglas de seguridad que permiten o deniegan el tráfico de red entrante o el tráfico de red saliente de varios tipos de recursos de Azure. Para cada regla, puede especificar un origen y destino, un puerto y un protocolo.

- Con un par de claves SSH puede crear una máquinas virtuales en Azure que usen claves SSH para la autenticación.

- Los discos administrados de Azure son volúmenes de almacenamiento de nivel de bloque que administra Azure y que se usan con Azure Virtual Machines. Los discos administrados se pueden considerar como un disco físico en un servidor local, pero virtualizado. Con los discos administrados, lo único que tiene que hacer es especificar el tamaño y el tipo del disco y aprovisionarlo. Cuando aprovisione el disco, Azure controla el resto.

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? 

- La conexión por shh funciona por sesión, en el momento en el que nos conectamos la sesión mantiene los procesos que ejecutamos en los recursos, sin esta todos ellos se terminan (también los subprocesos).

¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

- La regla sirve para que se indique el servicio que puerto estará usando , en este caso se usó el puerto 3000.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

Para el tamaño B1ls tenemos:

![image](https://user-images.githubusercontent.com/25957863/201828019-b593865c-b2ab-4281-888d-dd90d4a08b34.png)

Para el tamaño B2ms tenemos:

![image](https://user-images.githubusercontent.com/25957863/201827292-30c3d222-f74e-42d0-9f5e-e09403f16f48.png)

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

Para el tamaño B1ls tenemos:

![image](https://user-images.githubusercontent.com/25957863/201824864-3f44e4fb-f32a-4eff-94e4-9ef21b695f02.png)

Se llega a consumir el 99,8250 % de la CPU realizando las peticiones con Postman, lo que significa que la maquina no cuenta con los suficientes recursos para realizar las peticiones de forma concurrente.

Para el tamaño B2ms tenemos:

![image](https://user-images.githubusercontent.com/25957863/201828631-57de28fa-15f0-4458-9b70-e341002f0aad.png)

Se llega a consumir el 51,01 % de la CPU realizando las peticiones con Postman, lo que significa que la maquina cuenta con suficientes recursos para realizar las peticiones de forma concurrente.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:

Para el tamaño B1ls tenemos:

En este caso como podemos observar en la siguiente imagen, el tiempo de ejecucion realizado por las pruabas newman fue de 2 minutos con 58.3s y muestra fallos en 0 de las 10 peticiones realizadas.  Adicionalmente se puede observar la media del tiempo de respuesta que en este caso fueron 17,8s, de igual forma el tiempo minimo y maximo.

![image](https://user-images.githubusercontent.com/25957863/201825855-3724711e-400d-4848-a0b6-5b6dcd13784e.png)

Para el tamaño B2ms tenemos:

En este caso como podemos observar en la siguiente imagen, el tiempo de ejecucion realizado por las pruabas newman fue de 2 minutos con 55.8s, lo cual se redujo no tan considerablemente, esto por que al realizar el cambio de tamaño de la maquina a b2ms, esta consiguio mas RAM y mas almacenamiento. De igual forma muestra fallos en 4 de las 10 peticiones realizadas. Estos fallos se deben a que se genera una alta concurrencia. 

![image](https://user-images.githubusercontent.com/25957863/201828671-3178bd11-676c-4098-9348-2ea57260c582.png)

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

![image](https://user-images.githubusercontent.com/25957863/201830677-9b9c59e1-527c-4979-9ad6-3ea9593027e0.png)

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

No del todo, si bien se ayudan a bajar los tiempos de ejecución, estos tiempos se ven afectados por la mala optimización del código. Realmente no fue un cambio significativo, y la relacion costo beneficio de la maquina B1lS no se compara a la B2ms.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

Implicaria que se tenga que incurrir en posibles sobrecostos, esto debido a que no se hace una buena planificación de la infraestructura y los recursos que netamente se van usar. Como vimos en el apartado anterior las diferencias entre los tamaños `B2ms` y `B1ls` implican costos de $56 USD por hora de ejecución. 

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

Si, se evidenció una constante mejora ya que la maquina tuvo mejores recursos para dar una respuesta optima a las solicitudes hechas con postman. Pasó de tener un consumo del 98% al 51% la CPU de la maquina.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?



### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![image](https://user-images.githubusercontent.com/25957863/202032597-de234747-99b5-4b8d-94b5-ecc9b6aa8ed5.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![image](https://user-images.githubusercontent.com/25957863/202032695-40a4fae1-bd53-43fe-9b5e-4846a8596b92.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![image](https://user-images.githubusercontent.com/25957863/202032705-0123a548-e22f-4a0c-92d0-5d8762a6bf81.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![image](https://user-images.githubusercontent.com/25957863/202032741-f9e97c3f-797d-48c7-9ad9-8e9a77573064.png)

Una vez creado confimamos y damos aceptar:

![image](https://user-images.githubusercontent.com/25957863/202032764-45a20c45-30d0-4263-8228-b53282f2f8b5.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![image](https://user-images.githubusercontent.com/25957863/202032822-1373ac3d-53c4-4dfa-9a65-3428c242eebe.png)

![image](https://user-images.githubusercontent.com/25957863/202032842-d0bbf8ee-6d5c-4d5f-a565-7ea39e1fc629.png)

![image](https://user-images.githubusercontent.com/25957863/202032870-872fe87f-ab7f-49bb-9ff7-3090350dfafd.png)

![image](https://user-images.githubusercontent.com/25957863/202032893-751569d8-dc09-4fbd-8af4-0cadfaa92500.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![image](https://user-images.githubusercontent.com/25957863/202032924-ec82d568-fa37-4c63-bae7-e345cc4281f1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![image](https://user-images.githubusercontent.com/25957863/202032967-fa487d22-3398-4685-b9be-1dc7faa3a2b9.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![image](https://user-images.githubusercontent.com/25957863/202032990-4a001d79-f947-46ee-8a78-750cfb482168.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![image](https://user-images.githubusercontent.com/25957863/202033013-73aa8b84-da6f-41c8-8b9e-47a8d6cd98c8.png)

Realizamos los mismos pasos para cada una de las maquinas virtuales.

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

Realizamos para todas las maquinas lo siguiente:

![image](https://user-images.githubusercontent.com/25957863/202036656-1d5115cf-4776-43ba-ab91-17fe0a00e63e.png)

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

Probando :
![image](https://user-images.githubusercontent.com/25957863/202036694-c7b64539-f81a-4497-bdcb-a0bd09966482.png)
![image](https://user-images.githubusercontent.com/25957863/202036723-e3fde105-d5e7-450d-bec7-6b5ac51e1bc3.png)


2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.



3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.




