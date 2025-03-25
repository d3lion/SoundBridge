<div align="center">
    <h1>🎵 SoundBridge 🎵</h1>
</div>

<div align="center">
    <img src="https://i.pinimg.com/originals/00/46/96/004696ce8a252df48f2f9ca667101a63.gif" alt="SoundBridge" width="800" height="200">
</div>

## 📜 Descripción

SoundBridge es un proyecto desarrollado para ayudar a una chica con sordera a recibir sonidos directamente en su cerebro. Utiliza una Raspberry Pi para captar los sonidos a través de un micrófono y luego los transmite a los audífonos de conducción ósea conectados por Bluetooth a un dispositivo que envía estas señales al cerebro.

## 🛠️ Componentes

- **🖥️ Raspberry Pi Zero 2W**: El cerebro del proyecto que gestiona la captura y transmisión de sonidos.
- **🎙️ Micrófono**: Captura los sonidos del entorno.
- **🎧 Audífonos de conducción ósea**: Emiten los sonidos que serán enviados al cerebro.
- **💻 Software**: Código desarrollado para gestionar la captura y transmisión de sonido.

## 📊 Diagrama

<div align="center">
    <img src="https://picockpit.com/raspberry-pi/wp-content/uploads/2021/10/zero-w-ports.jpg" alt="Diagrama de SoundBridge">
</div>

1. **💾 Micro SD Slot**: Aquí introducimos la tarjeta SD con el sistema operativo, instalado previamente en el ordenador, usar diferentes adaptadores según tus necesidades.
2. **🔌 Mini HDMI Port**: Aquí conectamos el cable HDMI normal a este puerto mediante un adaptador, lo que nos permite la visión por pantalla.
3. **🔗 Puerto MICRO USB**: Enchufamos los periféricos, micrófono, teclado, etc.
4. **🔋 Puerto PWR IN**: Este puerto suministra energía a la placa, manteniéndola encendida, puede estar enchufada a una batería portátil, al ordenador, o al enchufe mediante un transformador de un cargador.

## ⚙️ Instalación

1. **Instalar Raspberry Imager**:
    - **Ubuntu**: [Imager para Ubuntu](https://downloads.raspberrypi.org/imager/imager_latest_amd64.deb)
    - **Windows**: [Imager para Windows](https://downloads.raspberrypi.org/imager/imager_latest.exe)
    - **MacOS**: [Imager para MacOS](https://downloads.raspberrypi.org/imager/imager_latest.dmg)
      
      ```bash
      sudo apt install rpi-imager
      ```

2. **Seleccionar SO**:
    - Una vez abierta la aplicación tenemos tres ventanas:
      - **Elegir Dispositivo**: En esta ventana elegimos las Raspberry Pi Zero 2W.
      - **Elegir Sistema Operativo**: En esta ventana deslizamos abajo y elegimos lo siguiente:
        
        Raspberry Pi OS(Other) > Raspberry Pi OS Lite(32 bits).
      - **Elegir Almacenamiento**: Elegimos la tarjeta SD.
    - No dar a siguiente todavía.

3. **Configurar SO**:
    - **Seleccionamos un hostname**: Aunque esto no es importante.
    - **Seleccionamos username y password**: Esto es importante recordar ya que siempre se va a iniciar sesión con estas credenciales.
    - **Configuramos la conexión a internet**: Si no tenemos pantalla, podemos hacer la conexión por SSH, para ello es importante que se introduzca la SSID del wifi y la contraseña.
    - **Habilitar conexión SSH**.
    - Se le puede dar a siguiente.

4. **Iniciar la Raspberry Pi y Configuración de red**:
    - Conectamos el cable de alimentación PWR IN, como se puede observar arriba en el diagrama, se inicia sesión, se comprueba si está conectado a una red wifi mediante:
      
      ```bash
      ping 8.8.8.8
      ```
      Esto debería mostrar que le llegan los paquetes.
    - Si no es así:
      
      ```bash
      raspberry-config
      ```
      Esto abrirá una interfaz a la que puedes ir a configuraciones de red y volver a intentar la conexión.

5. **Instalar dependencias**:
    ```bash
    sudo apt-get update & upgrade
    sudo apt-get install -y pulseaudio pulseaudio-module-bluetooth bluez pavucontrol
    sudo systemctl --user enable bluetooth
    sudo systemctl start bluetooth
    sudo systemctl --user enable pulseaudio
    sudo systemctl start pulseaudio
    pactl load-module module-bluetooth-discover
    ```
6. **Emparejar cascos bluetooth**:

   ```bash
   bluetoothctl
   scan on
   pair <MAC>
   connect <MAC>
   ```
7. **Conectar en pulseaudio**:
   ```bash
   pactl list sinks
   pactl list sources
   pactl load-module module-loopback source=MICROFONO sink=AURICULARES latency_msec=50
   ```

8. **Ajustes finales**:
   ```bash
   pavucontrol #Gestionar el sonido
   ```
   Si quieres que se ejecute al iniciar la raspberry siempre:
   ```
    ~/.config/pulse/default.pa
   ```
   y añade:
   ```
   pactl load-module module-loopback source=MICROFONO sink=AURICULARES latency_msec=50
   ```

   Final:
   ```
   pulseaudio -k
   pulseaudio --start
   ``
# **Mejoras del audio**
## 🎯 **Objetivos:**  
✅ **Subir el volumen del loopback** al máximo posible sin distorsionar.  
✅ **Evitar saturación** para que el sonido sea claro.  
✅ **Optimizar la latencia** para que el sonido sea lo más instantáneo posible.  

## 🛠 **Pasos a seguir:**

### 🔹 1. **Cargar el módulo Loopback si no está activo**
Si aún no lo has hecho, activa el loopback para capturar y reenviar el sonido:  
```bash
pactl load-module module-loopback latency_msec=1
```
💡 **Reducimos la latencia a 1ms** para evitar retrasos.

---

### 🔹 2. **Subir el volumen al máximo sin distorsión**  

#### 📌 **Aumentar el volumen del Loopback**
Primero, encuentra el **ID del loopback**:  
```bash
pactl list sink-inputs | grep -E 'Sink Input|Volume'
```
Verás algo como:  
```
Sink Input #42
    Volume: front-left: 65536 / 100% / 0 dB, front-right: 65536 / 100% / 0 dB
```
Aquí, el **ID es 42**. Ahora súbelo más del 100% (con cuidado de la distorsión):  
```bash
pactl set-sink-input-volume 42 150%
```
Si se sigue escuchando bajo, prueba **200%**:
```bash
pactl set-sink-input-volume 42 200%
```

---

#### 📌 **Aumentar el volumen de los auriculares Bluetooth**  
Encuentra el ID de los auriculares:  
```bash
pactl list sinks | grep -E 'Sink #|Name|Volume'
```
Si el ID es **1**, sube su volumen:  
```bash
pactl set-sink-volume 1 150%
```

---

### 🔹 3. **Reducir la saturación con un compresor de audio**  
Si el sonido empieza a distorsionar al aumentar el volumen, podemos aplicar un **compresor** con **`ladspa`**, que ajustará el volumen automáticamente.  

Primero, instala el paquete necesario:  
```bash
sudo apt install swh-plugins
```
Luego, carga el módulo con el **compresor**:  
```bash
pactl load-module module-ladspa-sink sink_name=compresor plugin=sc4_1882 label=sc4 control=5,1,200,-20,10,5,12
```
📌 **Explicación de los parámetros**:  
- `5` → Tiempo de ataque rápido (reacciona rápido a cambios de volumen).  
- `1` → Tiempo de liberación corto.  
- `200` → Ratio del compresor (reduce picos de volumen).  
- `-20` → Umbral de compresión (suaviza los sonidos fuertes).  
- `10` → Ganancia de compensación (recupera el volumen).  
- `5` → Evita distorsión.  
- `12` → Máximo nivel de salida.  

Ahora, redirige el audio del **Loopback** al nuevo **"compresor"**:  
```bash
pactl move-sink-input 42 compresor
```

---

### 🔹 4. **Hacer que los cambios sean permanentes**  
Si todo funciona bien, haz que los cambios se carguen al iniciar **PulseAudio**.  

Edita el archivo de configuración:  
```bash
nano ~/.config/pulse/default.pa
```
Añade al final:  
```
load-module module-loopback latency_msec=1
load-module module-ladspa-sink sink_name=compresor plugin=sc4_1882 label=sc4 control=5,1,200,-20,10,5,12
```
Guarda con `CTRL + X`, `Y`, y `ENTER`.

---

## 🎧 **Resumen Final**
1️⃣ **Activa el Loopback** → Captura y reenvía el sonido.  
2️⃣ **Sube el volumen del loopback** → `pactl set-sink-input-volume <ID> 200%`  
3️⃣ **Sube el volumen del Bluetooth** → `pactl set-sink-volume <ID> 150%`  
4️⃣ **Añade un compresor de audio** → Evita la saturación.  
5️⃣ **Haz los cambios permanentes** → Para que se carguen siempre.  

## 🚀 Uso

1. Iniciar la Raspberry Pi Zero 2W.
2. Ejecutar el software siguiendo las instrucciones de instalación.
3. El micrófono captará los sonidos y los audífonos de conducción ósea los reproducirán.

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Puedes hacerlo a través de Pull Requests o creando Issues para reportar problemas o sugerir mejoras.

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo `LICENSE` para más detalles.

## 📧 Contacto

Para cualquier duda o consulta, por favor contacta a jose.garcia220501@gmail.com.
