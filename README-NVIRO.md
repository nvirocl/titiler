
# TiTiler

Fork del proyecto `TiTiler` (una aplicación que permite servir raster tiles construida en python con FastApi) para nuestra aplicación de Nviro Monitor.  

---

## Instalación

Para construir y levantar el docker ejecutar desde la raiz:

```
docker compose up --build titiler
```

---
## Convertir TIFF a COG

Para aprovechar las bondades de TiTiler, se debe transformar el raster a COG (que es un TIFF enfocado en la nube), para transformarlos se puede utilizar gdal con el siguiente comando:

```
gdal_translate -of COG \
  -co COMPRESS=DEFLATE \
  -co BLOCKSIZE=512 \
  -co BIGTIFF=IF_SAFER \
  -co OVERVIEW_RESAMPLING=NEAREST \
  input.tif output_cog.tif
```

## Ubicación de los COG

Los rasters en formato COG deben ser almacenados en la carpeta configurada en el `docker-compose.yml` en:

```
services:
  titiler:
    volumes:
      - ./:/data
```

## API

Se puede revisar la api  de TiTiler con los endpoints disponibles entrando a la ruta:

```
localhost:8000/api.html
```

## Documentación

https://developmentseed.org/titiler/


## Producción

### Prerequisitos

* Tener instalado `uv`

* Instalar `cdk`, con el siguiente comando puedes instalarlo:

```bash
npm install -g aws-cdk@^2
```

Para desplegar en producción, se debe utilizar el stack de AWS CDK que se encuentra en la carpeta `deployment/aws/`.
Además ya que se realizaron cambios en el código base hay que generar packages locales para que luego sean cargados con el comando `cdk deploy`:

El mismo proyecto presenta un readme para desplegar en producción el cual está presente en https://developmentseed.org/titiler/deployment/aws/lambda/#deploy aunque a mi no me resultó así que dejaré el proceso más adelante documentado.

Antes de partir asegúrate de tener un archivo `.env` configurado correctamente en la carpeta `deployment/aws/` con las variables de entorno necesarias. Puedes usar el archivo `.env.example` de la misma carpeta como referencia.

En resumen, se tienen los siguientes comandos:

- Si ocurren cambios en el código base, será necesario hacer un rebuild del proyecto principal antes de hacer el deploy. Esto se hace en la ruta principal con:

  ```bash
  docker compose -f docker-compose-for-build.yml up --build
  ```

  Con esto quedarán generados archivos `.whl` en la carpeta `deployment/aws/lambda/dist/` que luego serán utilizados en el despliegue.
- Luego, para desplegar en producción, se debe ejecutar desde `deployment/aws` (si ya has pasado por el 1er deploy):

  ```bash
  cdk deploy
  ```
  Al correr este comando se hará un build para soportar lambda con los paquetes generados o preexistentes del paso anterior. Luego de esto se procederán a mostrar los cambios y se solicitará confirmación para proceder con el despliegue. **Nota que este proceso puede tardar un par de minutos debido a la construcción de la imagen de lambda y la carga de los paquetes.**


### Primer deploy

Adjunto los comandos que me funcionaron, en la documentación oficial del repo se mencionaban con `uv` pero estos no funcionaron.

```bash
# prepara la cuenta de aws para recibir despliegues con cdk
cdk bootstrap

# pre-genera el template de cloudformation
cdk synth

# despliega el stack en aws
cdk deploy
```
