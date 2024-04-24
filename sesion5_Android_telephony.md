## Inicio

1. Crear un nuevo proyecto
    > File > new > New Project > empty views activity

2. Crear una segunda actividad
    > File > new > activity > empty views activity (la llamamos TelephonyActivity)
    
    > activity_telephony.xml:

        - Añadimos un textView text="Telephony Activity" id=telephonyTV
        - Añadimos un boton    text="Obtener Informacion de Red" id="getNetInfoBTN
        - Añadimos un boton    text="Obtener Informacion de la celda" id="getCellInfoBTN


3. Enlazamos la actividad principal con "TelephonyActivity"
    > main_activity.java:
    ```java
    public void openTelephonyActivity(View v)
    {
        long clickTime = System.currentTimeMillis();
        Intent intent = new Intent(this, TelephonyActivity.class);
        intent.putExtra("timestamp", clickTime);
        startActivity(intent);
    }
    ```
    > activity_main.xml:

        - Añadimos un boton text="Telephony" id="goToTelephonyBTN"
        - Enlazamos el boton con el método anterior onClick="openTelephonyActivity"

4. Recibimos los datos que hemos enviado desde la actividad principal
    > TelephonyActivity.java
    ```java

    // Definimos esta funcion para extraer el timestamp del intent
    protected Date getTimestampFromIntent(Bundle extras)
    {
        if (!extras.isEmpty())
        {
            Object extra = extras.get("timestamp");
            if (extra instanceof Long)
            {
                return new Date ((Long) extra);

            }
        }
        return null;
    }

    // Hacemos uso de dicha funcion en OnCreate()
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_telephony);

        // Retrieve exchanged information
        Bundle extras = getIntent().getExtras();
        Date timestamp = getTimestampFromIntent(extras);
        if (timestamp == null)
        {
            Log.e("TelephonyActivity", "Error while retrieving the timestamp");
        }
        else
            Log.i("TelephonyActivity", "Timestamp correctly received: " + timestamp.toString());

    }
    ```

## Configuracion para la obtención de datos de telefonía

1. Permisos
    > AndroidManifest.xml
    ```xml
        <!-- Para acceder a cierta información de teléfonía necesitaremos tener activa la ubicación -->
        <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
        <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    ```

2. Creamos una instancia del cliente que nos dará acceso a los servicios de telefonía
    ```java
    // TelephonyActivity.java

    private TelephonyManager telephonyManager;

    // Inicialización en el metodo onCreate()
    this.telephonyManager = (TelephonyManager) getSystemService(TELEPHONY_SERVICE);
    ```

3. Añadimos los métodos para obtener información sobre el dispositivo
    ```java
    // TelephonyActivity.java

    public void getNetInfo(View v) {
        showTerminalInfo();
    }

    public void showTerminalInfo() {
        // Chequeamos permisos
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_PHONE_STATE) != PackageManager.PERMISSION_GRANTED) {
            if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.READ_PHONE_STATE)) {
                Log.i("TelephonyActivity", "Should show rationale");
            } else {
                ActivityCompat.requestPermissions(this, new String[]
                        {
                                Manifest.permission.READ_PHONE_STATE
                        }, 0);
            }
            return;
        }

        String text = "";
        // Obtenemos el tipo de red para comunicaciones de voz
        text += this.telephonyManager.getVoiceNetworkType() + " ";
        // Obtenemos el tipo de red para comunicaciones de datos
        text += this.telephonyManager.getDataNetworkType() + " ";
        // Obtenemos el nombre del operador de red
        text += this.telephonyManager.getNetworkOperatorName() + " ";

        TextView textView = findViewById(R.id.telephonyTV);
        textView.setText(text);
    }
    ```

    **NOTA**: Es necesario comprobar que se han dado los permisos necesarios o solicitarlos al usuario

    ```java
    // TelephonyActivity.java

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        switch (requestCode)
        {
            // Request permissions for ShowTerminalInfo()
            case 0:
            {
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED)
                {
                    Log.d("LocationAction", "Permisos de acceso a datos de telefonia del terminal concedidos");
                    showTerminalInfo();
                }
                else {
                    Log.e("LocationAction", "Permisos de acceso a datos de telefonia del terminal DENEGADOS");
                }
            }
        }
    }

    ```

4. Enlazamos el botón de la vista Telephony con el método getNetInfo()

## Recolección de datos de la celda
De igual modo, podemos acceder a información sobre las celdas de telefonía. Para ello, vamos a definir un método para obtener dicha informacion, **showCellInfo()**, y otro que se ejecutará
al pulsar el boton, **getCellInfo()**

1. AÑadimos los dos métodos

```java
    // TelephonyActivity.java

    public void getCellInfo(View v) {
        showCellInfo();
    }

    public void showCellInfo() {
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.ACCESS_FINE_LOCATION)) {
                Log.i("TelephonyActivity", "Should show rationale");
            } else {
                ActivityCompat.requestPermissions(this, new String[]
                        {
                                Manifest.permission.ACCESS_FINE_LOCATION
                        }, 1);
            }
            return;
        }

        List<CellInfo> cellInfoList = telephonyManager.getAllCellInfo();
    }
```

2. Dado que también se requiere de permisos, añadiremos el codigo 1 al método **onRequestPermissionsResult()** para poder diferenciar la seleccion de permisos de este metodo con la del anterior:

```java
    // TelephonyActivity.java
    //      > onRequestPermissionsResult()

    case 1:
    {
        if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED)
        {
            Log.d("LocationAction", "Permisos de acceso a datos de celda del terminal concedidos");
            ();
        }
        else {
            Log.e("LocationAction", "Permisos de acceso a datos de celda del terminal DENEGADOS");
        }
    }

```

2. Enlazamos el boton con el método getCellInfo()

3. En este punto, nuestro objeto cellInfoList contiene un lista de elementos tipo CellInfo que pueden ser de **distintas tecnologías**. Por eso, tendremos que tratar cada caso verificando el tipo de tecnología al que pertenece. Añadimos entonces lo siguiente a nuestra funcion showCellInfo():

```java
        StringBuilder text = new StringBuilder();
        for (CellInfo info : cellInfoList)
        {
            // Celda 3G
            if (info instanceof CellInfoWcdma)
            {

            }
            // celda 4G
            else if (info instanceof CellInfoLte)
            {

            }
            // celda 5G
            else if (info instanceof CellInfoNr)
            {

            }
            else
                Log.e("TelephonyActivity", "Cell information not from 3/4/5G");
        }
```

> **3G**

```java
    // Celda 3G
    if (info instanceof CellInfoWcdma)
    {
        CellInfoWcdma cellInfoWcdma = (CellInfoWcdma) info;
        CellIdentityWcdma id = cellInfoWcdma.getCellIdentity();
        text.append("WCDMA_ID: {cid: ").append(id.getCid());
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.P)
        {
            text.append(" mcc: ").append(id.getMcc());
            text.append(" mnc: ").append(id.getMnc());
        } else
        {
            text.append(" mcc: ").append(id.getMccString());
            text.append(" mnc: ").append(id.getMncString());
        }
        // Location Area Code
        text.append(" lac: ").append(id.getLac());
        // Signal Strength 
        text.append(" level: ").append(cellInfoWcdma.getCellSignalStrength().getLevel()).append("\n");
    }
```

**NOTA:** la potencia de la señal se puede obtener de diferentes formas. En este caso se hace uso de **getLevel()**, la cual nos devuelve la potencia como un valor entero dentro de un rango definido [SignalStrength](https://developer.android.com/reference/android/telephony/SignalStrength#getLevel()).

> **4G**
```java
    // celda 4G
    else if (info instanceof CellInfoLte)
    {
        CellInfoLte cellInfoLte = (CellInfoLte) info;
        CellIdentityLte id = cellInfoLte.getCellIdentity();
        // Cell Identity
        text.append("LTE_ID: {cid: ").append(id.getCi());
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.P)
        {
            text.append(" mcc: ").append(id.getMcc());
            text.append(" mnc: ").append(id.getMnc());
        } else
        {
            text.append(" mcc: ").append(id.getMccString());
            text.append(" mnc: ").append(id.getMncString());
        }
        text.append(" tac: ").append(id.getTac());
        text.append("} level: ").append(cellInfoLte.getCellSignalStrength().getLevel()).append("\n");
    }
```

> 5G NR
```java
    // celda 5G
    else if (info instanceof CellInfoNr)
    {
        CellInfoNr cellInfoNr = (CellInfoNr) info;
        CellIdentityNr id = (CellIdentityNr) cellInfoNr.getCellIdentity();
        text.append("NR_ID: {cid: ").append(id.getNci());
        text.append(" mcc: ").append(id.getMccString());
        text.append(" mnc: ").append(id.getMncString());
        text.append(" tac: ").append(id.getTac());
        text.append("} level: ").append(cellInfoNr.getCellSignalStrength().getLevel()).append("\n");
    }
```

## Información a tener en cuenta 

- En nuestro terminal físico podemos establecer (y por tanto limitar) la tecnología de red a la que queramos que nuestro terminal se conecte (3G/4G/5G)
- Al intentar rescatar información de la celda puede que algún parámetro muestre el valor 2147483647, lo que nos indica que esta información no está disponible (CellInfo.UNAVAILABLE)
