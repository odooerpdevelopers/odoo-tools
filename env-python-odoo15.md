## Guia instalación entorno python para Odoo 15 (Linux, macOS)

__ Esta guía puede ser usada como base y ejemplo para poner en marchar otras versiones como odoo 14 __

Odoo 15 usa librerias como pyopenssl, cryptography, gevent en unas versiones
que en sistemas como macOs, Windows y otras distros de Linux 
puede que no sean soportadas, o tengan conflictos
con esas versiones especificas, este script es un patch para solucionar y crear
un entorno python para odoo 15 exitoso
¡¡¡Importante!!!: reemplaza la direccion del proyecto donde este tu odoo15 
(la carpeta /repositorio de github que contiene odoo-bin)

**Requisitos:** Tener instalado Conda o miniconda, tener descargado el repositorio de Odoo 15, Git

**Sistemas compatibles:** (Linux, macOS), si tienes Windows puede que te ayude tambien en gran medida

Descargar o clonar Odoo desde: https://github.com/odoo/odoo/tree/15.0

Atencion!!! Si haces la instalacion con el script odoo15 no haria falta descargar odoo.

Como instalar Miniconda: https://docs.anaconda.com/miniconda/install/#quick-command-line-install

Nota final: el fichero result.txt excluira las librerias cryptography, gevent, greenlet
pero deja pyopenssl, al instalar pyopenssl instalará automaticamente cryptography
y buscara la version mas actual, lo mismo pasa cuando se instala gevent con greenlet


## Ruta directorio Odoo 15
``` bash
# creamos una variable de entorno con la ruta de odoo 15
# esto hace facil localizarla y llamarla desde cualquier lugar en el terminal
# reemplaza la direccion con la ruta de tu sistema
export odoo15_path="/Users/jcmontoya/odoo_dev/odoo15/odoo"
```

## Creando un entorno python con Conda

``` bash
# creamos un entorno con conda para odoo 15
conda create --name conda_odoo15 python=3.9

# activa el entorno
conda activate conda_odoo15
```

### Actualizamos entorno pip
```bash
pip install --upgrade setuptools pip wheel
```

### Esta linea elimina las lineas con las libs conflictivas del requirements.txt de odoo y crea un nuevo fichero result.txt

```bash
sed -E '/^cryptography|^greenlet|^gevent/d' $odoo15_path/requirements.txt > $odoo15_path/result.txt

# debes dirigirte antes a la ruta donde esta el proyecto de odoo descargado de github 
# para localizar el fichero requirements.txt, o indicarle el path completo en el terminal
# ejemplo: sed -E '/^cryptography|^greenlet|^gevent/d' /odoo_path/requirements.txt > /odoo_path/result.txt
```

### Instalando las librerias pip del fichero nuevo result.txt
```bash
pip install -r $odoo15_path/result.txt
# eliminamos cryptography por defecto que tiene conflictos con pyopenssl 19.0.0
# instalamos una compatible 3.4.6 con odoo 15
pip uninstall cryptography
pip install cryptography==3.4.6
# instalamos gevent y greenlet (al instalar gevent instala automaticamente greenlet como dependencia)
pip install gevent
# Listo!!! ahora ya deberias tener un entorno existoso sin fallos para odoo 15
```

### desactivar el entorno conda
```bash
conda deactivate
```

### Operaciones entorno Conda
```bash
# muestra todos los entornos creados
conda env list
# eliminando un entorno conda por nombre
conda env remove --name conda_odoo15
```
