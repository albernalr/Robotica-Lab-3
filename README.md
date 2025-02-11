# Laboratorio No. 03 - 2024-II - Robótica de Desarrollo, Introducción a ROS

Este repositorio contiene el desarrollo paso a paso del Laboratorio No. 03 de Robótica de Desarrollo, enfocado en la introducción a ROS (Robot Operating System). A continuación, se detallan los requisitos y resultados de aprendizaje.

## Requisitos

1. **Instalación de ROS2 Humble**: Sigue el tutorial disponible en el repositorio del laboratorio. [Link al repositorio](https://github.com/fegonzalez7/rob_unal_clase2).
2. **MATLAB 2015b o superior**: Asegúrate de tener MATLAB instalado en tu equipo.
3. **Robotics Toolbox de MathWorks**: Disponible desde la versión 2015 en adelante.
4. **Tutoriales de ROS2**: Son una excelente fuente de ayuda. Puedes encontrarlos en la [wiki oficial de ROS](http://wiki.ros.org/ROS/Tutorials).

## Resultados de Aprendizaje

1. **Conocer y explicar los conceptos básicos de ROS**: Entender cómo funciona el sistema operativo robótico y sus componentes principales.
2. **Usar los comandos fundamentales de ROS**: Familiarizarse con los comandos básicos de ROS para la gestión de nodos, tópicos y servicios.
3. **Conectar nodos de ROS con MATLAB**: Aprender a integrar MATLAB con ROS para la simulación y control de robots.
4. **Realizar conexión inicial de motores Dynamixel con ROS**: Configurar y controlar motores Dynamixel utilizando ROS.

## Ejercicios en el Laboratorio

### a) Familiarizarse con los comandos de Linux

Antes de comenzar con ROS, es importante familiarizarse con los comandos básicos de Linux. Algunos de los comandos más utilizados son:

- `ls`: Lista los archivos y directorios en el directorio actual.
- `cd`: Cambia de directorio.
- `mkdir`: Crea un nuevo directorio.
- `rm`: Elimina archivos o directorios.
- `sudo`: Ejecuta un comando con privilegios de superusuario.
- 
### b) Conexión de ROS2 con MATLAB

#### Procedimiento:

1. **Iniciar el nodo maestro de ROS**:
   - Abrir una terminal en Linux y ejecuta el siguiente comando:
     ```bash
     roscore
     ```
   - Esto iniciará el nodo maestro de ROS, que es necesario para la comunicación entre nodos.

2. **Ejecutar el nodo de turtlesim**:
   - En una segunda terminal, ejecutar el siguiente comando:
     ```bash
     rosrun turtlesim turtlesim_node
     ```
   - Esto iniciará la simulación de la tortuga en turtlesim.

3. **Conectar MATLAB con ROS**:
   - Abrir MATLAB en Linux (asegúrate de tener el Robotics Toolbox instalado).
   - Crea un script en MATLAB con el siguiente código para suscribirte al tópico de la pose de la tortuga:
     ```matlab
     % Crear un suscriptor al tópico de la pose de la tortuga
     poseSub = rossubscriber('/turtle1/pose', 'turtlesim/Pose');

     % Obtener el último mensaje de la pose
     poseData = receive(poseSub, 10); % Espera hasta 10 segundos para recibir un mensaje

     % Mostrar la posición y orientación de la tortuga
     disp('Posición de la tortuga:');
     disp([poseData.X, poseData.Y]);
     disp('Orientación de la tortuga:');
     disp(poseData.Theta);
     ```

4. **Enviar comandos a la tortuga**:
   - Para modificar la pose de la tortuga, se puede utilizar los servicios de turtlesim. Por ejemplo, para mover la tortuga a una posición específica, se puede usar el servicio `turtle1/teleport_absolute`.

5. **Finalizar el nodo maestro**:
   - Para finalizar el nodo maestro en MATLAB, se puede usar el siguiente comando:
     ```matlab
     rosshutdown
     ```

### c) Utilizando Python

#### Procedimiento:

1. **Crear un script de Python para controlar la tortuga**:
   - En el paquete `hello_turtle` de ROS2, dentro de la carpeta de scripts, crea un archivo llamado `myTeleopKey.py`.
   - Escribir el siguiente código para controlar la tortuga con el teclado:
     ```python
     #!/usr/bin/env python3

     import rospy
     from geometry_msgs.msg import Twist
     from turtlesim.srv import TeleportAbsolute, TeleportRelative

     def move_turtle():
         rospy.init_node('turtle_teleop_key', anonymous=True)
         pub = rospy.Publisher('/turtle1/cmd_vel', Twist, queue_size=10)
         rate = rospy.Rate(10)  # 10hz

         while not rospy.is_shutdown():
             key = input("Presiona una tecla (W/A/S/D/R/ESPACIO): ")

             twist = Twist()

             if key == 'w':
                 twist.linear.x = 1.0  # Mover hacia adelante
             elif key == 's':
                 twist.linear.x = -1.0  # Mover hacia atrás
             elif key == 'a':
                 twist.angular.z = 1.0  # Girar en sentido antihorario
             elif key == 'd':
                 twist.angular.z = -1.0  # Girar en sentido horario
             elif key == 'r':
                 # Teleportar a la posición central
                 rospy.wait_for_service('/turtle1/teleport_absolute')
                 try:
                     teleport_abs = rospy.ServiceProxy('/turtle1/teleport_absolute', TeleportAbsolute)
                     teleport_abs(5.5, 5.5, 0)  # Posición central
                 except rospy.ServiceException as e:
                     print("Service call failed: %s" % e)
             elif key == ' ':
                 # Girar 180 grados
                 rospy.wait_for_service('/turtle1/teleport_relative')
                 try:
                     teleport_rel = rospy.ServiceProxy('/turtle1/teleport_relative', TeleportRelative)
                     teleport_rel(0, 3.14159)  # Girar 180 grados
                 except rospy.ServiceException as e:
                     print("Service call failed: %s" % e)

             pub.publish(twist)
             rate.sleep()

     if __name__ == '__main__':
         try:
             move_turtle()
         except rospy.ROSInterruptException:
             pass
     ```

2. **Incluir el script en el archivo `CMakeLists.txt`**:
   - Asegúrate de incluir el script `myTeleopKey.py` en el apartado `catkin_install_python` del archivo `CMakeLists.txt`.

3. **Compilar el paquete**:
   - Dirígete al directorio del workspace de catkin y ejecuta el siguiente comando:
     ```bash
     catkin_make
     ```

4. **Ejecutar el script**:
   - Abre tres terminales:
     - En la primera terminal, ejecuta `roscore`.
     - En la segunda terminal, ejecuta `rosrun turtlesim turtlesim_node`.
     - En la tercera terminal, ejecuta `rosrun hello_turtle myTeleopKey.py`.
   - Ahora podrás controlar la tortuga con las teclas W, A, S, D, R y ESPACIO.

## Actividades

1. **Resumen de la instalación de ROS2**:
   - Describe los pasos principales que seguiste para instalar ROS2 en tu sistema operativo (Ubuntu, Robostack, WSL).
   - Menciona las dificultades que encontraste durante la instalación o el arranque.

2. **Ejercicios iniciales**:
   - Muestra los ejercicios realizados con scripts de MATLAB y/o Python, incluyendo capturas de pantalla o videos de los resultados.
   - Utiliza herramientas como `rviz` o `rosbag` para visualizar los datos.

3. **Uso de Dynamixel Wizard**:
   - Describe los parámetros de motor utilizados, los comandos ejecutados y los resultados obtenidos. Incluye fotos o videos del proceso.

## Observaciones

- **Repositorio en GitHub**: Crea un repositorio en GitHub donde expliques la metodología, los resultados, los análisis y las conclusiones del laboratorio.
- **Forma de trabajo**: Trabaja en grupos de laboratorio.
- **Comentarios en el código**: Asegúrate de incluir comentarios en las funciones implementadas y adjunta los archivos `.m` y `.py` en el repositorio.

## Entrega

- **Forma de trabajo**: Grupal de 2 personas. Cada integrante debe proporcionar la URL del repositorio creado.
- **Entregables**: Crea un repositorio en GitHub con los resultados y análisis del laboratorio.
- **Fecha de entrega**: Consulta la actividad en Moodle para conocer la fecha límite de entrega.

## Referencias

- [ROS Tutorials](http://wiki.ros.org/ROS/Tutorials)
- [Robotics - UNAL - LAB2](https://github.com/fegonzalez7/rob_unal_clase2)
- [Hello Turtle](https://github.com/felipeg17/hello_turtle)
