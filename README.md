# GUÍA DE COMANDOS PARA ECOSISTEMA ROS1/ROS2 CON DOCKER

<div align="center">

<a href="https://www.docker.com/" target="_blank">
<img src="https://github.com/devicons/devicon/blob/master/icons/docker/docker-original.svg" title="Docker" alt="Docker" width="40" height="40"/>
</a>
<a href="http://www.ros.org/" target="_blank">
<img src="https://upload.wikimedia.org/wikipedia/commons/b/bb/Ros_logo.svg" title="ros" alt="ros" width="80" height="40"/>
</a>
<a href="https://www.linux.org/" target="_blank">
<img src="https://github.com/devicons/devicon/blob/master/icons/linux/linux-original.svg" title="linux" alt="linux" width="40" height="40"/>
</a>
<a href="https://ubuntu.com/" target="_blank">
<img src="https://github.com/devicons/devicon/blob/master/icons/ubuntu/ubuntu-plain.svg" title="Ubuntu" alt="Ubuntu" width="40" height="40"/>
</a>

</div>

## INSTALACIÓN Y CONFIGURACIÓN INICIAL

```bash
# Requisitos previos
sudo apt update
sudo apt install -y docker.io docker-compose xterm

# Añadir usuario al grupo Docker (evita usar sudo para comandos docker)
sudo usermod -aG docker $USER
# IMPORTANTE: Cerrar sesión y volver a iniciar para aplicar cambios de grupo

# Clonar repositorio y entrar al directorio (si aplica)
# git clone https://tu-repositorio.git
# cd tu-directorio

# Permitir conexiones X11 desde Docker (necesario para aplicaciones gráficas)
xhost +local:docker

# Construir las imágenes (sólo primera vez o al actualizar Dockerfiles)
docker-compose build

# Iniciar todos los contenedores
docker-compose up -d
```

## GESTIÓN DE CONTENEDORES

```bash
# Ver estado de contenedores
docker-compose ps

# Iniciar contenedores (si están detenidos)
docker-compose start

# Detener contenedores (mantiene datos)
docker-compose stop

# Detener y eliminar contenedores (útil para reiniciar desde cero)
docker-compose down

# Reiniciar todos los contenedores
docker-compose restart

# Ver logs (útil para diagnóstico)
docker-compose logs

# Ver logs de un contenedor específico en tiempo real
docker-compose logs -f ros_bridge
```

## ACCESO A LOS CONTENEDORES

```bash
# Entrar al contenedor ROS1 Melodic (abre un terminal interactivo)
docker exec -it ros1_melodic bash

# Entrar al contenedor ROS2 Humble
docker exec -it ros2_humble bash

# Entrar al contenedor del bridge ROS1-ROS2
docker exec -it ros_bridge bash

# Ejecutar un comando directamente en un contenedor
docker exec ros1_melodic [comando]
# Ejemplo: docker exec ros1_melodic rostopic list
```

## COMUNICACIÓN ROS1-ROS2 CON BRIDGE

```bash
# Verificar que el bridge esté funcionando
docker-compose logs ros_bridge

# Listar tópicos de ROS1 (desde el contenedor ROS1)
docker exec -it ros1_melodic bash -c "source /opt/ros/melodic/setup.bash && rostopic list"

# Listar tópicos de ROS2 (desde el contenedor ROS2)
docker exec -it ros2_humble bash -c "source /opt/ros/humble/setup.bash && ros2 topic list"

# Publicar un mensaje de prueba en ROS1
docker exec -it ros1_melodic bash -c "source /opt/ros/melodic/setup.bash && rostopic pub /test std_msgs/String 'data: \"Mensaje desde ROS1\"' -r 1"

# Escuchar el mensaje en ROS2 (debería recibir los mensajes de ROS1)
docker exec -it ros2_humble bash -c "source /opt/ros/humble/setup.bash && ros2 topic echo /test"

# Publicar un mensaje de prueba en ROS2
docker exec -it ros2_humble bash -c "source /opt/ros/humble/setup.bash && ros2 topic pub /test2 std_msgs/String 'data: \"Mensaje desde ROS2\"' -r 1"

# Escuchar el mensaje en ROS1
docker exec -it ros1_melodic bash -c "source /opt/ros/melodic/setup.bash && rostopic echo /test2"
```

## EJEMPLOS PRÁCTICOS

### Ejemplo 1: TurtleSim entre contenedor y host

```bash
# Iniciar turtlesim en ROS1
docker exec -d ros1_melodic bash -c "source /opt/ros/melodic/setup.bash && rosrun turtlesim turtlesim_node"

# Controlar turtlesim desde una nueva ventana de terminal
xterm -e "docker exec -it ros1_melodic bash -c 'source /opt/ros/melodic/setup.bash && rosrun turtlesim turtle_teleop_key'"

# Alternativamente, iniciar turtlesim en ROS2
docker exec -d ros2_humble bash -c "source /opt/ros/humble/setup.bash && ros2 run turtlesim turtlesim_node"

# Controlar turtlesim ROS2 desde una nueva ventana
xterm -e "docker exec -it ros2_humble bash -c 'source /opt/ros/humble/setup.bash && ros2 run turtlesim turtle_teleop_key'"
```

### Ejemplo 2: Comunicación entre ROS1 en contenedor y ROS2 en host

```bash
# Desde el host (Ubuntu 22.04 con ROS2 instalado)
# Primero, configurar variables de entorno
export ROS_DOMAIN_ID=0
source /opt/ros/humble/setup.bash

# Publicar un mensaje desde el host a ROS1
ros2 topic pub /mensaje_host std_msgs/String '{data: "Mensaje desde el host"}' -r 1

# Escuchar en ROS1 (contenedor)
docker exec -it ros1_melodic bash -c "source /opt/ros/melodic/setup.bash && rostopic echo /mensaje_host"
```

## DISPOSITIVOS USB Y SERIALES

```bash
# Listar dispositivos USB conectados (desde el host)
lsusb

# Listar puertos seriales disponibles (desde el host)
ls -l /dev/ttyUSB* /dev/ttyACM*

# Verificar puertos seriales desde los contenedores
docker exec -it ros1_melodic ls -l /dev/ttyUSB* /dev/ttyACM*
docker exec -it ros2_humble ls -l /dev/ttyUSB* /dev/ttyACM*

# Cambiar permisos de un dispositivo si es necesario
sudo chmod 666 /dev/ttyACM0
```

### Ejemplo: Conexión con Arduino en ROS1

```bash
# Cargar código Arduino compatible con rosserial previamente

# Iniciar nodo serial para Arduino en ROS1
docker exec -it ros1_melodic bash -c "source /opt/ros/melodic/setup.bash && rosrun rosserial_python serial_node.py _port:=/dev/ttyACM0 _baud:=57600"

# Verificar tópicos publicados por Arduino
docker exec -it ros1_melodic bash -c "source /opt/ros/melodic/setup.bash && rostopic list | grep arduino"
```

### Ejemplo: Conexión con microcontrolador en ROS2

```bash
# Instalar micro-ROS si es necesario
docker exec -it ros2_humble bash -c "apt update && apt install -y ros-humble-micro-ros-agent"

# Conectar con un dispositivo usando micro-ROS
docker exec -it ros2_humble bash -c "source /opt/ros/humble/setup.bash && ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyACM0 -b 115200"
```

## GESTIÓN DE ESPACIOS DE TRABAJO Y PAQUETES

### ROS1 (Melodic)

```bash
# Compilar espacio de trabajo ROS1
docker exec -it ros1_melodic bash -c "cd /ros1_ws && source /opt/ros/melodic/setup.bash && catkin_make"

# Crear un nuevo paquete ROS1
docker exec -it ros1_melodic bash -c "cd /ros1_ws/src && source /opt/ros/melodic/setup.bash && catkin_create_pkg mi_paquete std_msgs rospy roscpp"
```

### ROS2 (Humble)

```bash
# Compilar espacio de trabajo ROS2
docker exec -it ros2_humble bash -c "cd /ros2_ws && source /opt/ros/humble/setup.bash && colcon build --symlink-install"

# Crear un nuevo paquete ROS2
docker exec -it ros2_humble bash -c "cd /ros2_ws/src && source /opt/ros/humble/setup.bash && ros2 pkg create --build-type ament_cmake mi_paquete_ros2"

# Compilar un paquete específico
docker exec -it ros2_humble bash -c "cd /ros2_ws && source /opt/ros/humble/setup.bash && colcon build --packages-select mi_paquete_ros2"
```

## SOLUCIÓN DE PROBLEMAS COMUNES

### Problema 1: No se pueden ver aplicaciones gráficas

```bash
# En el host:
xhost +local:docker

# Verificar configuración de DISPLAY en el contenedor
docker exec -it ros1_melodic bash -c "echo $DISPLAY"

# Asegurarse de que las variables de entorno X11 estén configuradas correctamente
docker-compose down
# Editar docker-compose.yml y verificar sección environment y volumes
docker-compose up -d
```

### Problema 2: El bridge no funciona

```bash
# Reiniciar solo el contenedor del bridge
docker-compose restart bridge

# Verificar logs del bridge
docker-compose logs -f bridge

# Asegurarse de que ROS1 Master está en ejecución
docker exec -it ros1_melodic bash -c "source /opt/ros/melodic/setup.bash && rostopic list"

# Si es necesario, reiniciar el roscore manualmente
docker exec -it ros1_melodic bash -c "source /opt/ros/melodic/setup.bash && roscore"
```

### Problema 3: Dispositivos USB no detectados

```bash
# Verificar que el dispositivo está conectado al host
ls -l /dev/ttyACM* /dev/ttyUSB*

# Actualizar permisos del dispositivo
sudo chmod 666 /dev/ttyACM0

# Reiniciar los contenedores después de conectar dispositivos
docker-compose restart

# Si el problema persiste, detenga los contenedores y modifique docker-compose.yml:
# - Asegúrese de tener privileged: true
# - Agregue el device específico bajo la sección devices
```

### Problema 4: Problemas de compilación

```bash
# Limpiar build y devel folders en ROS1
docker exec -it ros1_melodic bash -c "rm -rf /ros1_ws/build /ros1_ws/devel"

# Limpiar build en ROS2
docker exec -it ros2_humble bash -c "rm -rf /ros2_ws/build /ros2_ws/install"

# Actualizar rosdep
docker exec -it ros1_melodic bash -c "rosdep update"
docker exec -it ros2_humble bash -c "rosdep update"

# Instalar dependencias faltantes para un workspace
docker exec -it ros1_melodic bash -c "cd /ros1_ws && rosdep install --from-paths src --ignore-src -r -y"
```

## COMANDOS ÚTILES ADICIONALES

```bash
# Guardar cambios en contenedor como nueva imagen
docker commit ros1_melodic mi_ros1_personalizado:v1

# Copiar archivos del host al contenedor
docker cp archivo.txt ros1_melodic:/ruta/destino/

# Copiar archivos del contenedor al host
docker cp ros1_melodic:/ruta/origen/archivo.txt ./

# Ver uso de recursos
docker stats

# Explorar imagen de contenedor (útil para depuración)
docker exec -it ros1_melodic bash -c "find / -name \"*nombre_archivo*\" 2>/dev/null"

# Limpiar recursos no utilizados (imágenes, contenedores parados, etc.)
docker system prune
```

## USAR ROS DESDE APLICACIONES EXTERNAS

```bash
# Permitir conexiones externas a ROS (para aplicaciones como RViz o rqt en el host)
# En el host, configurar variables de entorno para ROS1:
export ROS_MASTER_URI=http://localhost:11311
export ROS_IP=127.0.0.1

# O para ROS2:
export ROS_DOMAIN_ID=0

# Lanzar RViz desde el host conectado al contenedor ROS1 (si ROS1 está instalado en el host)
rviz

# O para ROS2:
ros2 run rviz2 rviz2
```

---

