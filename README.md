# SO101-Imitation-learning-for-stacking-cups

**Materia:** Implementación de Robótica Inteligente - TE3002B.102
**Profesor:** Nezih Nieto Gutiérrez
**Institución:** Tecnológico de Monterrey

## Descripción general

Este es un proyecto de robótica cuyo objetivo es entrenar un robot *SO-101* para realizar una tarea de manipulación de vasos mediante *Imitation Learning*.

El sistema utiliza una configuración *leader-follower*, donde un robot SO-101 leader es manipulado por un operador y un robot SO-101 follower replica los movimientos mientras se graban demostraciones. Estas demostraciones se usan posteriormente para entrenar una política que permite al robot ejecutar la tarea de forma autónoma.

La tarea principal consiste en mover y reorganizar vasos sobre una mesa usando observaciones visuales de dos cámaras y el estado articular del robot.

<img width="1600" height="1200" alt="SO101_setup (1)" src="https://github.com/user-attachments/assets/e414aa79-7780-4ef4-a557-0cc29fee102c" />

---

## Objetivo del proyecto

El objetivo principal del proyecto fue desarrollar una política de control capaz de manipular vasos de forma autónoma usando datos reales recolectados con el robot SO-101.

### Objetivos específicos
- Construir y calibrar dos robots SO-101.
- Configurar un sistema leader-follower para teleoperación.
- Preparar un workspace experimental controlado.
- Grabar datasets de demostraciones usando LeRobot.
- Entrenar una política de Imitation Learning.
- Probar el modelo entrenado en el robot físico.

---

## Hardware utilizado

Desde el inicio, fue necesario construir físicamente los robots y preparar el entorno experimental:

### Robots

- 1 robot **SO-101 leader/master** para teleoperación.
- 1 robot **SO-101 follower/slave** para ejecutar la tarea.
- Piezas impresas en 3D (para los robots).
- Servomotores.
- Tornillería.
- Cables de alimentación y comunicación.
- Cámaras USB.
- Computadora con GPU.

Para la lista de materiales completa y guia de armado, nos referimos al siguiente repositorio en GitHub: https://github.com/TheRobotStudio/SO-ARM100

### Workspace

- Placa de madera para superficie de trabajo.
- Vasos para la tarea de manipulación.
- Cámaras colocadas en posiciones fijas.
- Iluminación controlada.
- Posiciones iniciales consistentes para los vasos.
- Área libre para los robots para evitar colisiones.

## Software utilizado

- Ubuntu 22.04
- Python
- LeRobot
- Hugging Face Hub
- PyTorch
- CUDA
- OpenCV
- Git
- tmux
- NVIDIA tools
 
---

## Arquitectura general del sistema

```text
Robot leader
    ↓ teleoperación
Robot follower
    ↓ ejecución física
Cámaras + estado articular
    ↓ grabación
Dataset en formato LeRobot
    ↓ entrenamiento
Modelo de Imitation Learning
    ↓ inferencia
Robot follower ejecutando la tarea autónomamente
```

Durante la grabación, el operador mueve el robot leader. El robot follower copia esos movimientos y ejecuta la tarea con los vasos. Al mismo tiempo, el sistema guarda imágenes, estados del robot y acciones.

Después, con el entrenamiento, el modelo aprende a imitar las acciones del operador a partir de las observaciones grabadas.

---

## Método de aprendizaje

El proyecto utiliza *Imitation Learning*, específicamente *Behaviour Cloning*.

El modelo aprende una función que relaciona:

```text
Observación actual → Acción correcta
```

Donde la observación incluye:

- Imagen de cámara frontal.
- Imagen de cámara lateral.
- Estado articular del robot.

Y la acción corresponde al movimiento que debe ejecutar el robot en el siguiente instante.

Durante el entrenamiento, el modelo compara su acción predicha contra la acción real ejecutada por el operador. El error se calcula como *loss* y el modelo ajusta sus parámetros para reducir ese error.

---

## Modelo utilizado

El modelo principal probado físicamente fue:

```text
act_cup_shuffle_060k
```

Este modelo fue entrenado con un dataset de 100 episodios y logró ejecutar la tarea de manipulación de vasos de forma satisfactoria.

### Modelos considerados

| Modelo                   |       Dataset | Estado                                    |
| ------------------------ | ------------: | ----------------------------------------- |
| Modelo con 100 episodios | 100 episodios | Probado físicamente con buenos resultados |
| Modelo con 200 episodios | 200 episodios | Pendiente de evaluación física            |

---

## Descripción del dataset

Se generaron datasets usando *LeRobot* y se organizaron en *Hugging Face*. Cada episodio representa una demostración completa de la tarea.

Cada trayectoria incluye:

- Video de cámara frontal.
- Video de cámara lateral.
- Estado articular del robot.
- Acciones ejecutadas por el operador.
- Timestamps.
- Índice de episodio.
- Descripción de la tarea.

### Configuración de grabación

| Parámetro                    | Valor                       |
| ---------------------------- | --------------------------- |
| Resolución                   | 320x240                     |
| Cámaras                      | 2 USB                       |
| FPS de cámara                | 30 Hz                       |
| FPS del dataset              | 15 FPS                      |
| Duración máxima por episodio | 29.9 segundos               |

---

## Nota sobre frecuencia de grabación

Durante el desarrollo se realizaron pruebas con distintas configuraciones de cámara. En un primer intento de grabación de 100 episodios, el loop de grabación llegó a bajar aproximadamente a *2.4 Hz*, lo que provocó pérdida de información temporal y fallas en el dataset.

Esto significa que el sistema solo estaba capturando alrededor de 2 o 3 muestras por segundo, lo cual no era suficiente para representar correctamente el movimiento del robot, el acercamiento al vaso y el contacto físico.

Para corregirlo, se redujo la resolución a *320x240*, se mantuvieron las cámaras a *30 Hz* y se guardó el dataset a *15 FPS*. Con esta configuración, pudimos hacer una grabación más estable y útil para el entrenamiento.

---

## Instalación

### 1. Clonar LeRobot

```bash
git clone https://github.com/huggingface/lerobot.git
cd lerobot
```

### 2. Crear ambiente virtual

```bash
python3 -m venv ~/lerobot-env
source ~/lerobot-env/bin/activate
```

### 3. Instalar LeRobot

```bash
pip install --upgrade pip
pip install -e ".[training]"
```

### 4. Instalar herramientas adicionales tmux/pynput

```bash
sudo apt update
sudo apt install -y v4l-utils tmux git
pip install pynput
```

### 5. Iniciar sesión en Hugging Face

```bash
hf auth login
hf auth whoami
```

---
 
Variables usadas durante el proyecto:

```bash
export MASTER_PORT=/dev/ttyACM0
export SLAVE_PORT=/dev/ttyACM1
export MASTER_ID=so101_master
export SLAVE_ID=so101_slave
export CAM_FRONT=0
export CAM_SIDE=2
export HF_USER=Ubuntus-x-Repo
```

## Grabación del dataset

Ejemplo de comando para grabar un dataset:

```bash
export DATASET_NAME=so101_cup_shuffle_$(date +%Y%m%d_%H%M%S)
export REPO_ID=$HF_USER/$DATASET_NAME

lerobot-record \
  --robot.type=so101_follower \
  --robot.port=$SLAVE_PORT \
  --robot.id=$SLAVE_ID \
  --robot.cameras="{ front: {type: opencv, index_or_path: $CAM_FRONT, width: 320, height: 240, fps: 30, fourcc: MJPG}, side: {type: opencv, index_or_path: $CAM_SIDE, width: 320, height: 240, fps: 30, fourcc: MJPG} }" \
  --teleop.type=so101_leader \
  --teleop.port=$MASTER_PORT \
  --teleop.id=$MASTER_ID \
  --display_data=true \
  --dataset.repo_id=$REPO_ID \
  --dataset.num_episodes=100 \
  --dataset.fps=15 \
  --dataset.episode_time_s=30 \
  --dataset.reset_time_s=120 \
  --dataset.single_task="Shuffle three cups on the table." \
  --dataset.push_to_hub=false
```

## Subir dataset a Hugging Face

Después de grabar localmente:

```bash
export DATASET_NAME=[NOMBRE_DEL_DATASET]
export REPO_ID=$HF_USER/$DATASET_NAME
export LOCAL_DIR=$HOME/.cache/huggingface/lerobot/$REPO_ID

hf repo create "$REPO_ID" --type dataset --exist-ok
hf upload "$REPO_ID" "$LOCAL_DIR" . --repo-type dataset
```

---

## Entrenamiento

### Entrenar modelo ACT

```bash
source ~/lerobot-env/bin/activate
cd ~/lerobot

lerobot-train \
  --policy.type=act \
  --dataset.repo_id=[HF_USER/DATASET_100_EPISODIOS] \
  --batch_size=8 \
  --steps=60000 \
  --output_dir=outputs/train/act_cup_shuffle_060k \
  --job_name=act_cup_shuffle_060k \
  --policy.device=cuda \
  --policy.push_to_hub=false \
  --wandb.enable=false \
  --log_freq=10 \
  --save_freq=5000
```

### Significado de métricas

| Métrica  | Significado                        |
| -------- | ---------------------------------- |
| `step`   | Paso de entrenamiento              |
| `smpl`   | Muestras procesadas                |
| `ep`     | Episodio o progreso interno        |
| `epch`   | Épocas completadas                 |
| `loss`   | Error de entrenamiento             |
| `grdn`   | Norma del gradiente                |
| `lr`     | Learning rate                      |
| `updt_s` | Tiempo de actualización del modelo |
| `data_s` | Tiempo de carga de datos           |

---

### Seguridad durante evaluación

Antes de ejecutar el modelo:

- Verificar que el robot esté en posición inicial.
- Mantener manos lejos del área de trabajo.
- Confirmar que las cámaras estén conectadas.
- Confirmar que el robot follower esté calibrado.
- Tener listo `Ctrl+C` o estar listo para apagar la corriente.
- Tener forma de cortar alimentación en caso de emergencia.
- Probar primero con objetos alejados o sin vasos si es necesario.

---

## Resultados actuales

El modelo entrenado con *100 episodios* logró ejecutar la tarea de manipulación de vasos de forma satisfactoria en pruebas físicas:

- El robot inició la trayectoria correctamente.
- El robot se acercó al vaso.
- El robot logró contacto físico con el vaso.
- El robot pudo mover el vaso.
- La ejecución fue similar a las demostraciones del dataset.
- El sistema funcionó de manera autónoma sin teleoperación durante la prueba.

---

## Métricas

# Entrenamiento

El modelo fue entrenado con el dataset de 100 episodios con 150,000 steps.

### Configuración y rendimiento general

| Métrica                       |        Resultado |
| ----------------------------- | ---------------: |
| Modelo                        |          SmolVLA |
| Episodios del dataset         |              200 |
| Steps acumulados              |          150,000 |
| Batch size                    |                8 |
| Muestras procesadas estimadas |        1,200,000 |
| Épocas completadas            |            20.45 |
| Tiempo total de entrenamiento |      13 h 45 min |
| Velocidad promedio            |     3.03 steps/s |
| Throughput promedio           | 24.24 muestras/s |

> Las muestras procesadas se estimaron multiplicando los 150,000 steps por un batch size de 8.

### Métricas de aprendizaje

| Métrica                                   |    Resultado |
| ----------------------------------------- | -----------: |
| Loss inicial registrada                   |        1.226 |
| Loss final                                |        0.154 |
| Loss mínima registrada                    |        0.128 |
| Loss promedio de los últimos 20 registros |      0.15945 |
| Gradient norm final                       |        1.420 |
| Learning rate final                       | `2.5 × 10⁻⁶` |
| Tiempo de actualización                   | 0.317 s/step |
| Tiempo de carga de datos                  | 0.007 s/step |

---

### Interpretación de resultados

La pérdida disminuyó de `1.226` a `0.154`, lo que representa una reducción aproximada del **87.4 %**.

```text
Reducción de loss = (1.226 - 0.154) / 1.226 ≈ 87.4 %
```

Esta reducción indica que el modelo mejoró considerablemente su capacidad para predecir las acciones realizadas por el operador en las demostraciones.

La loss mínima fue de `0.128`, mientras que la loss final fue de `0.154`. Esta diferencia es normal, ya que cada registro se calcula usando minibatches distintos y algunos pueden contener movimientos más complejos que otros. Por esta razón, el promedio de los últimos registros, `0.15945`, ofrece una representación más estable del comportamiento final.

El modelo procesó aproximadamente 1.2 millones de muestras, equivalentes a cerca de 20.45 recorridos completos del dataset. Esto permitió que las demostraciones fueran utilizadas varias veces durante el proceso de optimización.

El gradient norm final fue de `1.420`, por debajo del límite de gradient clipping configurado en `10.0`. Esto sugiere que las actualizaciones de los parámetros permanecieron controladas y no presentaron una explosión sostenida de gradientes.

El learning rate terminó en `2.5 × 10⁻⁶`. Este valor bajo indica que, durante las etapas finales, el modelo realizaba ajustes pequeños y precisos en lugar de modificaciones grandes que pudieran desestabilizar el aprendizaje.

El tiempo de carga de datos fue de solo `0.007 segundos` por step, mientras que la actualización del modelo tomó aproximadamente `0.317 segundos`. Por lo tanto, el principal costo computacional se encontró en el procesamiento y actualización de la red neuronal, no en la lectura del dataset.

---

### Conclusión del entrenamiento

Los resultados indican que el modelo convergió sobre el dataset de entrenamiento:

- La loss disminuyó significativamente.
- Los gradientes permanecieron controlados.
- El learning rate se redujo progresivamente.
- La carga de datos fue eficiente.
- El modelo completó los 150,000 steps planeados.

El modelo entrenado con 100 episodios logró ejecutar la tarea de forma satisfactoria, por lo que concluimos que el entrenamiento fue estable y que si se adapta al mundo real, por lo que con este proyecto demostramos que es posible entrenar un robot SO-101 real para realizar una tarea de manipulación de vasos usando demostraciones humanas e Imitation Learning.

---

## Links importantes

### Datasets

| Dataset               | Link                         |
| --------------------- | ---------------------------- |
| Dataset 100 episodios | https://huggingface.co/datasets/Ubuntus-x-Repo/so101_cup_shuffle_100_base_20260607_133944 |
| Dataset 200 episodios | https://huggingface.co/datasets/Ubuntus-x-Repo/so101_cup_shuffle_200_base_20260607_190216 |

### Videos

| Evidencia           | Link         |
| ------------------- | ------------ |
| Demo y setup        | https://drive.google.com/drive/folders/1Z9In9e2dTJbLXM12zIxB2GG1G8nZI6Ng?usp=sharing |
| Cámaras / grabación | (grabaciones visibles en links de Hugging Face)                                      |

---

## Limitaciones actuales

- El modelo depende de condiciones similares al entrenamiento.
- Cambios grandes en cámaras o iluminación pueden afectar resultados.
- La tarea depende de fricción y contacto físico.
- No existe todavía una política de recuperación.
- El modelo de 200 episodios queda pendiente de evaluación física.
- Las métricas cuantitativas formales quedan pendientes.

---

## Trabajo futuro

- Probar físicamente el modelo entrenado con 200 episodios.
- Comparar modelo de 100 episodios contra modelo de 200 episodios.
- Medir tasa de éxito en múltiples intentos.
- Entrenar modelos con 300, 500 o más episodios.
- Incluir más variación en posiciones iniciales.
- Probar diferentes ángulos de cámara.
- Mejorar la superficie de trabajo.
- Agregar detección explícita de vasos.
- Implementar una política de recuperación.
- Comparar ACT contra SmolVLA u otras arquitecturas.

---

## Integrantes del equipo y contribuciones

| Integrante                 | Contribuciones principales                                                                    |
| -------------------------- | --------------------------------------------------------------------------------------------- |
| José Omar Cavazos Gonzalez | Compró materiales, realizó el setup físico del sistema y grabó 100 episodios para el dataset. |
| Angel Dominguez            | Propuso la idea principal del proyecto e investigó el uso de LeRobot para el entrenamiento.   |
| Pablo Aguilar              | Apoyó en la calibración del robot y grabó 50 episodios para el dataset.                       |
| Sebastian Toro             | Imprimió las piezas 3D necesarias para el armado del robot SO-101.                            |
| Fernando Proal             | Apoyó en el setup, calibración del sistema y grabó 200 episodios para el dataset.             |
| Manuel Ferro               | Sin contribución específica reportada para esta versión del proyecto.                         |
| Alejandro Kurt             | Apoyó en el setup del sistema y en la lluvia de ideas inicial del proyecto.                   |
| Zacbe Ortega               | Apoyó en el segundo proceso de entrenamiento del modelo.                                      |
| Fabricio Banda             | Grabó 100 episodios utilizados para el entrenamiento del modelo.                              |
