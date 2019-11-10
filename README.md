# AdminPro

# Rutas
En este proyecto se ha utilizado un archivo principal de rutas llamado app.routes.ts
En este archivo se encuentran las rutas "principales" que utilizará la página y estas rutas serán controladas por el <router-outlet> del archivo app.component.html

La idea es modularizar todo lo posible, por ello se crean mas archivos de rutas que van a ir relacionando los componentes, servicios y elementos compartidos.
Es decir, se crea un archivo de rutas para todos los componentes de "pages" --> pages.routes.ts . En este archivo van a ir todas las rutas (que serán rutas hijas) de todos los componentes que se vayan añadiendo en la carpeta "pages". 

Para poder separar la parte "principal" de la página de la parte "hija" (los componentes) se crea un archivo llamado "pages.component.html" el cual tendra otro <router-outlet>. La diferencia entre este y el de app.component.html es que este, el de pages, va a controlar las rutas hijas de pages.routes.ts


# Modulos
Al igual que las rutas, los modulos también deben ir segmentados.
El programa va a tener un archivo de modulos principal llamado app.module.ts y en él deben ir los modulos principales y se van a importar los modulos "hijos" (estos no son hijos como tal como si lo son las rutas)

Hay que crear un modulo para pages, otro para los servicios, otro para shared...

Por ejemplo en el pages.module.ts se va a añadir (en imports) el archivo de rutas de pages, para que luego cuando se importe pages.module.ts en app.module.ts el archivo de rutas de app pueda controlar al archivo de rutas hijas de pages

# Servicios
Se ha creado un archivo donde se guardan todas las rutas de todos los servicios, y exportandolo podemos usar ese archivo para poder importarlos en cualquier componente de la aplicación, y en caso de modificaciones o nuevos servicios basta con escribirlo en el service.index.ts.

También se crea un modulo para todos los servicios, así luego en el app.module.ts solo hay que añadir el módulo para tener disponibles todos los servicios.

# Atributos Personalizados
Hay atributos en HTML que no pueden ser vinculados a una variable del TS, la primera solución a esto es crear Atributos Personalizados como se ve en el siguiente código.

Archivo HTML:
<div class="progress">
    <!-- aria-valuenow es la propiedad que controla el punto de progreso de la barra de progreso.
        Por defecto no se le puede vincular a una variable de ts, saltaria un error de "Error: Template parse"
        En estos casos se puede añadir [attr.nombrePropiedad]="variable" y ahora si se puede controlar. NOTA: si la propiedad esta separa con "-" aqui se pone una mayuscula solo -->
    <div class="progress-bar progress-bar-striped progress-bar-animated" role="progressbar" [attr.ariaValuenow]="progreso"          aria-valuenow="50" aria-valuemin="0" aria-valuemax="100" [style.width]="progreso + '%'">
    </div>
</div>

Archivo TS:
    progreso: number = 50

# @Input y @Output
Para que un componente hijo pueda recibir información desde el padre hay que realizar lo siguiente:

    1: Escribir la palabra clave '@Input()' delante de la declaración de la variable en el componente hijo. Ver: incrementador.component.ts
    2: Al llamar al componente hijo desde el html del componente padre hay que escribir el mismo nombre de la variable del ts y el valor que va a tener la variable (<app-incrementador leyenda="mi leyenda"></app-incrementador>). Ver: progress.component.html

Para que un componente hijo pueda enviar información al padre hay que trabajar con eventos utilizando el @Output
Un breve resumen de lo que habría que hacer:

    1: En el componente HIJO hay que declara el Output: @Output() nombreFuncion: EventEmitter<tipoDato> = new EventEmitter();
    2: En el componente HIJO hay que indicar en algún lado cuál es el valor que se envía. Por ejemplo si tenemos una función que calcule un valor lo suyo sería al final de la función poner algo como esto -> this.nombreFuncion.emit( this.variableAEnviar ); Ver: incrementador.component.ts
    3: En el componente PADRE hay que indicar dónde se va a recibir la información enviada por el hijo. En la declaracion del componente hijo (<app-hijo></app-hijo>) se le añade algo como lo siguiente -> (nombreFuncion)="variableQueCambia = $event". Ver progress.component.html

    NOTA: en este caso es solo una variable lo que se cambia, si fuera algo más complejo lo normal es crear una función (en el PADRE) para trabajar con los datos recibidos por el HIJO -> (nombreFuncion)="metodo($event)". 

# ViewChild
La forma de vincular un componente HTML con el TS es utilizando @ViewChild().
En el componente HTML hay que poner un indicador para el elemento que se quiere vincular al ts, utilizando "#". Por ejemplo <input #txt>
Ahora en el TS lo que hay que hacer es declarar el ViewChild de la siguiente forma -> @ViewChild('txt', {static:false}) variable: ElementRef
"variable" es la variable con la que se va a trabajar en TS que es la relacionada con el elemento HTML

Si por ejemplo (en este caso que es un input) se quiere cambiar el valor que tiene el input sería -> this.variable.nativeElement.value = this.progreso; "progreso" es una variable cualquiera con un valor cualquiera

Si se quiere tener el foco en el input con el que se está interactuando sería -> this.variable.nativeElement.focus();

Ver incrementador.component

# Forma alternativa a ViewChild para referenciar un componente HTML desde TS
Existe otra forma de hacer lo mismo que con ViewChild.
Lo primero que hay que hacer es, en el constructor del archivo TS escribir lo siguiente:
    constructor(@Inject(DOCUMENT) private _document) { }

Con esto ya se puede utilizar la varibale _document para referenciar a un componente HTML.
Por ejemplo nos creamos un metodo que cambie el estilo css en función de un string:
    cambiarColor( color: string ) {
        url: string
        this._document.getElementById('tema').setAttribute('href', url);
    }

En este ejemplo la única forma de decir que "_document" es un elemento html en concreto es buscándolo por el "id", en este caso "tema". En el componente html que se quiere referenciar habrá que poner este id. IMPORTANTE: EL ID NO DEBE ESTAR REPETIDO EN NINGÚN OTRO COMPONENTE HTML
    <link id=tema>

De esta forma da igual dónde se encuentre el componente html y donde esté el ts (respecto a la organización de las carpetas), siempre va a poder ser referenciado.

Ver settings.service.ts e index.html



Añadido: También se puede pasar por parámetro de una función un componente html.
Para ello en el HTML hay que poner un indicador con "#" (como se hace con ViewChild) y en el evento (click) (o el que sea) al indicar la función se pone como parámetro el indicador -> (click)="funcion(indicador)"
Ver account-setting.component

# Cambio dinámico de Tema CSS y persistencia de ajustes
Ver settings.service.ts y account-setting.component para ver como se ha realizado el cambio de temas Css y la persistencia de la información con el LocalStorage