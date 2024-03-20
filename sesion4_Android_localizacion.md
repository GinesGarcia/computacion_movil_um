## Autoría
    - Jesús García Rodríguez
    > Colaboradores
        - Ginés García Avilés
## Uso de localización en Android

1. Crear un nuevo proyecto
    > File > new > New Project > empty views activity

2. Crear una segunda actividad
    > File > new > activity > empty views activity (la llamamos LocationActivity)
    
    > activity_location.xml:

        - Añadimos un textView text="Segunda Actividad" id=secondActTV
        - Añadimos un boton    text="Obtener Localizacion" id="getLocationButton

3. Enlazamos la actividad principal con "LocationActivity"
    > main_activity.java:
    ```java
    public void openLocationActivity(View v)
    {
        Intent intent = new Intent(this, LocationActivity.class);
        startActivity(intent);
    }
    ```
    > activity_main.xml:

        - Añadimos un boton text="Ir a Localizacion" id="goToLocationButton"
        - Enlazamos el boton con el método anterior onClick="openLocationActivity"

## Configuracion para el uso de Localizacion

1. Permisos
    > AndroidManifest.xml
    ```xml
        <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
        <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    ```
2. Instalamos google play services (necesarios para localizacion)
    > Menu > tools > SDK Manager > SDK Tools

3. Añadimos dependencias en Gradle
    > Opcion 1: build.gradle.kts
    ```
        implementation("com.google.android.gms:play-services-location:21.2.0")
    ```
    > Opcion 2: File > Project Structure > dependencies
        - Al pulsar + podremos añadir las dependencias

## Localizacion en la aplicacion
1. Creamos una instancia del cliente que nos abstrae de los detalle de localizacion
    ```java
    // LocationActivity.java

    // Cliente de localizacion (atributo de la clase)
    private FusedLocationProviderClient client;

    // Inicialización en el metodo onCreate()
    this.client = LocationServices.getFusedLocationProviderClient(this);
    ```
2. Añadimos el método onClick() que será donde pongamos la lógica para acceder a la ubicacion
    ```java
    // LocationActivity.java


    public void onClick(View v) {
        this.client.getLastLocation();
    }
    ```

3. Ya tenemos los permisos en manifest por lo que nos permite usar getLastLocation()

4. Es necesario comprobar que efectivamente el usuario ha dado permisos. Aceptamos sugerencia que nos añade una comprobacion (if sentence)
    ```java
    // LocationActivity.java

    public void onClick(View v) {
        if (ActivityCompat.checkSelfPermission(this, android.Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED && 
        ActivityCompat.checkSelfPermission(this, android.Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            // TODO: Consider calling
            //    ActivityCompat#requestPermissions
            // here to request the missing permissions, and then overriding
            //   public void onRequestPermissionsResult(int requestCode, String[] permissions,
            //                                          int[] grantResults)
            // to handle the case where the user grants the permission. See the documentation
            // for ActivityCompat#requestPermissions for more details.
            return;
        }
        this.client.getLastLocation();
    }
    ```

5. Ahora, creamos la solicitud de los permisos al usuario siguiendo el TODO que nos indica el comentario del codigo generado
    ```java
    // LocationActivity.java

    ...
    import android.Manifest;
    ...

    if (ActivityCompat.checkSelfPermission...)
        ...

        // Solicitamos permisos al usuario ()
        ActivityCompat.requestPermissions(this, new String[]{
                Manifest.permission.ACCESS_FINE_LOCATION
        }, 0);

        ...

    ```

6. En este punto, es necesario gestionar la peticion que se genera cuando el usuario acepta (o no) el uso de los permisos. Para eso, y siguiendo la sugerencia en el comentario tras el todo, creamos la siguiente funcion:
    ```java
    // LocationActivity.java

    /**
     *
     * @param requestCode The request code passed in requestPermissions()
     * @param permissions The requested permissions. Never null.
     * @param grantResults The grant results for the corresponding permissions
     *     which is either {@link android.content.pm.PackageManager#PERMISSION_GRANTED}
     *     or {@link android.content.pm.PackageManager#PERMISSION_DENIED}. Never null.
     *
     */
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case 0: {
                // Si tenemos permisos
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    // TODO
                }
                // Si no los tenemos
                else {
                    Log.e("LocationAction", "Se necesitan permisos para poder usar la aplicacion");
                }
            }
        }
    }
    ```

7. Por claridad del código, vamos a poner el contenido de la funcion on click en una nueva funcion
    ```java
    // LocationActivity.java

        private void showLastLocation()
    {
        // Check permissions
        if (ActivityCompat.checkSelfPermission(this, android.Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED && ActivityCompat.checkSelfPermission(this, android.Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            // Request permissions to user (pop-up)
            ActivityCompat.requestPermissions(this, new String[]{
                    Manifest.permission.ACCESS_FINE_LOCATION
            }, 0);
            return;
        }
        this.client.getLastLocation();
    }
    ```

8. Por ultimo llamamos a showLastLocation() desde **onRequestPermissionsResult()** y desde **onClick()**
    ```java
    // LocationActivity.java

        ...

        // Si tenemos permisos
        if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            // Solicitamos la localizacion
            showLastLocation();
        }

        ...
    ```

## Recepción de la localizacion
el método client.getLastLocation() no nos devuelve una localizacion, sino un listener que será capaz de recuperar la informacion de localizacion cuando esté disponible.
Para manejar ese evento, hacemos lo siguiente:

1. En la funcion showLastLocation(), añadimos lo siguiente:

```java
    // LocationActivity.java

    private void showLastLocation()
    {
        ...

        this.client.getLastLocation().addOnSuccessListener(new OnSuccessListener<Location>() {
            @Override
            public void onSuccess(Location location) {
                // Aqui situaremos el código que necesita hacer uso de la localizacion
            }
        });

        ...
    }
```

2. En este ejemplo usaremos el dato de localizacion para mostrarlo en el textView de la actividad de localizacion. para eso añadimos lo siguiente en **showLastLocation()**:
    ```java
    // LocationActivity.java

    this.client.getLastLocation().addOnSuccessListener(new OnSuccessListener<Location>() {
        @Override
        public void onSuccess(Location location) {
            // Aqui situaremos el código que necesita hacer uso de la localizacion
            String text = location.getLatitude() + " " + location.getLongitude();
            TextView textView = findViewById(R.id.locationTV);
            textView.setText(text);
        }
    });
    ```

3. Por último, tendremos que hacer que nuestro boton llame a la función onClick(). Para ello, iremos a activity_location.xml, seleccionamos el botón y asignamos el método onClick() en el campo onClick de la seccion de atributos comunes (similar a lo que hacíamos al principio para hacer la transición entre actividades)

## Ejecucion
Antes de ejecutar, Añadir un punto de ubicacion
 - Menu de dispositivos virtuales activos > ... (extended controls) > location
 - Buscamos en el mapa un punto
 - Marcamos "Save Point"

## Resultado
- Al pulsar el boton para obtener la localización nos solicita permisos
- Si lo concedemos, nos aparece la latitud y la longitud en el cuadro de texto

**NOTA:** Si no nos pide los permisos, será necesario eliminar la aplicación del dispositivo virtual (la arrastramos a la papelera)

## Simulacion de la ubicacion
> Podemos simular distintas ubicaciones usando el emulador

    - Menu de dispositivos virtuales activos > ... (extended controls) > location

> El tipo de peticion que estamos usando (getLastLocation) no es activa, por lo que aunque cambiemos la ubicacion y pulsemos el boton no se actualiza (se toma la última).


## Solicitud activa de ubicacion
Ahora queremos tener acceso a datos de localizacion pero actualizados cada cierto tiempo.
Para ello nos tenemos que suscribir a un servicio que nos enviará actualizaciones de localizacion cada cierto tiempo.

1. Creamos e inicializamos el callback que recibirá las actualizaciones:
```java

// Attribute
LocationCallback locationCallback;

// onCreate()
this.locationCallback = new LocationCallback() {
    @Override
    public void onLocationResult(@NonNull LocationResult locationResult) {
        // super.onLocationResult(locationResult);
        // Lista de localizaciones (nos da las mejores dentro de los permisos que tengamos)
        Location location = locationResult.getLastLocation();
        String text = location.getLatitude() + " " + location.getLongitude();
        TextView textView = findViewById(R.id.locationTV);
        textView.setText(text);
    }
};
```

2. Creamos la solicitud de suscripcion al servicio de actualizacion de la localizacion añadiendo lo siguiente a la funcion **showLastLocation()**:
```java

    // Comentamos esto del apartado anterior
    /* First Part
    this.client.getLastLocation().addOnSuccessListener(new OnSuccessListener<Location>() {
        @Override
        public void onSuccess(Location location) {
            // Aqui situaremos el código que necesita hacer uso de la localizacion
            String text = location.getLatitude() + " " + location.getLongitude();
            TextView textView = findViewById(R.id.locationTV);
            textView.setText(text);
        }
    });
        */
    LocationRequest locationRequest = LocationRequest.create();
    locationRequest.setInterval(2000);
    locationRequest.setFastestInterval(1000);
    locationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);

    client.requestLocationUpdates(locationRequest, locationCallback, null);
```

3. Probamos que la ubicacion cambia al modificar nuestra localizacion usando el menu donde hemos guardado las localizaciones (debe cambiar si darle al boton porque estamos escuchando en el callback)

## Cuando paramos las peticiones de ubicacion?
Para no incurrir en un consumo excesivo de recursos, utilizaremos el metodo onPause() para parar las actualizaciones cuando nuestra aplicación no esté en uso.

1. metodo onPause
```java
@Override
protected void onPause() {
    super.onPause();
    client.removeLocationUpdates(locationCallback);
}
```

## Mapas
Para hacer uso de una vista de mapa con google maps, seguimos los siguientes pasos:

1. Añadimos un GoogleMapsView a nuestra aplicación
> File > new > Google > Google Maps View Activity

2. Introducimos nuestra clave generada en google  
link para generarla: https://developers.google.com/maps/documentation/android-sdk/get-api-key

```
# local.properties
MAPS_API_KEY=<key>
```
```xml
# AndroidManifest.xml
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="${MAPS_API_KEY}" />
```


En este punto ya tenemos integrado nuestro mapa pero a efectos prácticos, solo se podrá hacer uso una vez el mapa esté listo, es decir, tras la funcion onMapReady(). Solo al final de esa función, podremos incluir nuestro código (o llamada a función) que hará uso del mapa en sí:

1. En primer lugar, ponemos un nuevo boton en nuestra actividad principal para que nos lleve a la vista del mapa

5. que podemos hacer con maps? Obtener la posición actual del usuario

```java
    
    public void onMapReady(GoogleMap googleMap) {
        ...

        showPosition();

        ...
    }

    private void showPosition()
    {
        //coger la posicion actual del usuario.
        mMap.setMyLocationEnabled(true);
    }
```

6. Necesitaremos permisos para poder usar la localizacion (podemos reutilizar lo que hemos visto para showLastLocation en la actividad de localizacion)

```java
    private void showPosition() {
        // Check permissions
        if (ActivityCompat.checkSelfPermission(this, android.Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED && ActivityCompat.checkSelfPermission(this, android.Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            // Request permissions to user (pop-up)
            ActivityCompat.requestPermissions(this, new String[]{
                    Manifest.permission.ACCESS_FINE_LOCATION
            }, 0);
            return;
        }
        mMap.setMyLocationEnabled(true);
    }

    /**
     *
     * @param requestCode The request code passed in requestPermissions()
     * @param permissions The requested permissions. Never null.
     * @param grantResults The grant results for the corresponding permissions
     *     which is either {@link android.content.pm.PackageManager#PERMISSION_GRANTED}
     *     or {@link android.content.pm.PackageManager#PERMISSION_DENIED}. Never null.
     *
     */
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case 0: {
                // Si tenemos permisos
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    showPosition();
                }
                // Si no los tenemos
                else {
                    Log.e("LocationAction", "Se necesitan permisos para poder usar la aplicacion");
                }
            }
        }
    }
```

