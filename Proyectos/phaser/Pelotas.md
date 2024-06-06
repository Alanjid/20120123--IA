Inicializamos las variables
```
var w=500;
var h=400;

var jugador;
var fondo;
var bala;
var menu;
var cursors;

var key_der;
var key_izq;
var key_up;
var key_down;

var reset;
var velocidadBala;
var coordenada_X, coordenada_Y, dis_bala_jugador;

var estado_der;
var estado_izq;
var estado_arriba;
var estado_abajo;
var estado_esquina;

var jugadorPosInicial = { x: 230, y: 200 };
var balaPosInicial = { x: w-10, y: h-10 };

var timer = 0;
var regresar = true;

var nnNetwork , nnEntrenamiento, nnSalida, datosEntrenamiento=[];
var modoAuto = false, eCompleto=false;

var juego = new Phaser.Game(w, h, Phaser.CANVAS, '', { preload: preload, create: create, update: update, render:render});
```

Generamos la función preload para mandar llamadar los sprites y dejar precargados los datos
```
function preload() { 
    juego.load.image('fondo', 'assets/game/fondo.jpg');
    juego.load.spritesheet('mono', 'assets/sprites/altair.png', 32, 48);
    juego.load.image('menu', 'assets/game/menu.png');
    juego.load.image('bala', 'assets/sprites/purple_ball.png');
}
```

Generamos la función create, esta nos va a funcionar para agregar fisicas a los personajes, más aparte generamos la neurona
con 3 entradas que vendrian siendo la distancia entre el jugador y la bala
y 4 salidas, que vendrian siendo la posición a la que se tiene que mover, ya sea arriba, abajo, izquierda, derecha o esquinas

```
function create() { 
    cursors = this.input.keyboard.createCursorKeys(); 
    juego.physics.startSystem(Phaser.Physics.ARCADE); 
    juego.time.desiredFps = 30; 
    fondo = juego.add.tileSprite(0, 0, w, h, 'fondo');

    var cornerPositions = [{ x: 0, y: 0 }, { x: w, y: 0 }, { x: 0, y: h }, { x: w, y: h }];
    var randomCorner = cornerPositions[Math.floor(Math.random() * cornerPositions.length)];
    bala = juego.add.sprite(randomCorner.x, randomCorner.y, 'bala');
    jugador = juego.add.sprite(w / 2, h / 2, 'mono');

    juego.physics.enable(jugador);
    jugador.body.collideWorldBounds = true; // Evitar que el jugador salga de los limites del mundo NO MOVER JONATAHN!!!
    var corre = jugador.animations.add('corre', [8, 9, 10, 11]); 
    jugador.animations.play('corre', 10, true); 
    juego.physics.enable(bala); 
    bala.body.collideWorldBounds = true; 
    bala.body.bounce.set(1); 
    bala.body.velocity.set(450); 

    pausaL = juego.add.text(w - 100, 20, 'Pausa', { font: '20px Arial', fill: '#fff' }); 
    pausaL.inputEnabled = true; 
    pausaL.events.onInputUp.add(pausa, self); 
    juego.input.onDown.add(mPausa, self); 

    key_der = juego.input.keyboard.addKey(Phaser.Keyboard.RIGHT);
    key_izq = juego.input.keyboard.addKey(Phaser.Keyboard.LEFT);
    key_up = juego.input.keyboard.addKey(Phaser.Keyboard.UP);
    key_down = juego.input.keyboard.addKey(Phaser.Keyboard.DOWN);

    nnNetwork =  new synaptic.Architect.Perceptron(3,6,6,4);
    nnEntrenamiento = new synaptic.Trainer(nnNetwork); 
}
```

Generamos la función en la que hacemos el entrenamiento de la neurona con datos que se van a cargar más adelante

```
function enRedNeural(){ 
    nnEntrenamiento.train(datosEntrenamiento, {rate: 0.0003, iterations: 15000, shuffle: true});
}
```

Esta función va a recibir los parametros de entrada del entrenamiento para saber a donde enviar al personaje dependiendo
de la distancia entre la bala y el jugados
```
function datosDeEntrenamiento(param_entrada){ 
    nnSalida = nnNetwork.activate(param_entrada);

    // Calculamos el valor máximo y el segundo valor más alto de la salida
    var sortedOutput = nnSalida.slice().sort((a, b) => b - a);
    var maxValue = sortedOutput[0];
    var secondMaxValue = sortedOutput[1];

    // Definimos un umbral relativo como un porcentaje del valor máximo
    var umbralRelativo = 0.1 * maxValue;

    // Si la diferencia entre el valor máximo y el segundo valor más alto supera el umbral relativo, consideramos que la red ha tomado una decisión clara
    if (maxValue - secondMaxValue > umbralRelativo) {
        return nnSalida.indexOf(maxValue); // Devolvemos el índice del valor máximo
    } else {
        return 4; // No se ha tomado una decisión clara
    }
}
```

Con esta función ponemos el juego en pausa activando una de las banderas
```
function pausa() {
    juego.paused = true;
    menu = juego.add.sprite(w/2, h/2, 'menu');
    menu.anchor.setTo(0.5, 0.5);
}
```

Con esta función al momento de iniciar el juego en modo entrenamiento reiniciamos todos los valores de la red neuronal 
y que no exista un entrenamiento previo o ruido
```
function resetRedNeuronal() {
    nnNetwork = new synaptic.Architect.Perceptron(3, 6, 6, 4);
    nnEntrenamiento = new synaptic.Trainer(nnNetwork);
    datosEntrenamiento = [];
}
```

Con esta función detectamos en donde se da click para saber si reiniciar el juego en modo entrenamiento o modo manual.
Aparte así mismo mandamos a llamar el reinicio de variables o terminamos el entrenamiento de la neurona
```
function mPausa(event) {
    if (juego.paused) {
        var menu_x1 = w / 2 - 270 / 2,
            menu_x2 = w / 2 + 270 / 2,
            menu_y1 = h / 2 - 180 / 2,
            menu_y2 = h / 2 + 180 / 2;
        var mouse_x = event.x,
            mouse_y = event.y;
        if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 && mouse_y <= menu_y1 + 90) {
            eCompleto = false;
            resetRedNeuronal(); // Reiniciar la red neuronal
            modoAuto = false;
        } else if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 + 90 && mouse_y <= menu_y2) {
            if (!eCompleto) {
                enRedNeural();
                eCompleto = true;
            }
            modoAuto = true;
        }
        resetVariables();
        menu.destroy();
        juego.paused = false;
    }
}
```

Esta función nos sirve cuando iniciamos el juego, la implemtamos para que no haya ruido o sesgos en el entrenamiento
```
function resetVariables(){
    var cornerPositions = [{ x: 0, y: 0 }, { x: w, y: 0 }, { x: 0, y: h }, { x: w, y: h }];
    var randomCorner = cornerPositions[Math.floor(Math.random() * cornerPositions.length)];

    velocidadBala = 450;
    bala.body.velocity.x = velocidadBala;
    bala.body.velocity.y = -velocidadBala;
    bala.position.x = randomCorner.x; 
    bala.position.y = randomCorner.y;
    key_der.isDown = false;
    key_down.isDown=false;
    key_izq.isDown=false;
    key_up.isDown=false;
    jugador.position.x=230;
    jugador.position.y=200;
}
```

Las siguientes funciones las realizamos para mandar al personaje a coordenadas especificas en el mapa, aparte de activar o desactivar
banderas que seran las que nos diran en donde se encuentra el persoonaje y aparte para el entrenamiento de la neurona
```
function derecha() {
    jugador.position.x = 300;
    jugador.position.y = 200;
    estado_der = 1;
    estado_izq = 0;
    estado_arriba = 0;
    estado_abajo = 0;
    estado_esquina = 0;
    regresar = false;
}

function izquierda() {
    jugador.position.x = 130;
    jugador.position.y = 200;
    estado_der = 0;
    estado_izq = 1;
    estado_arriba = 0;
    estado_abajo = 0;
    estado_esquina = 0;
    regresar = false;
}

function arriba() {
    jugador.position.x = 230;
    jugador.position.y = 120;
    estado_der = 0;
    estado_izq = 0;
    estado_arriba = 1;
    estado_abajo = 0;
    estado_esquina = 0;
    regresar = false;
}

function abajo() {
    jugador.position.x = 230;
    jugador.position.y = 280;
    estado_der = 0;
    estado_izq = 0;
    estado_arriba = 0;
    estado_abajo = 1;
    estado_esquina = 0;
    regresar = false;
}

function esquinaArribaIzquierda() {
    jugador.position.x = 130;
    jugador.position.y = 120;
    estado_der = 0;
    estado_izq = 0;
    estado_arriba = 0;
    estado_abajo = 0;
    estado_esquina = 1;
    regresar = false;
}

function esquinaArribaDerecha() {
    jugador.position.x = 300;
    jugador.position.y = 120;
    estado_der = 0;
    estado_izq = 0;
    estado_arriba = 0;
    estado_abajo = 0;
    estado_esquina = 2;
    regresar = false;
}

function esquinaAbajoIzquierda() {
    jugador.position.x = 130;
    jugador.position.y = 280;
    estado_der = 0;
    estado_izq = 0;
    estado_arriba = 0;
    estado_abajo = 0;
    estado_esquina = 3;
    regresar = false;
}

function esquinaAbajoDerecha() {
    jugador.position.x = 300;
    jugador.position.y = 280;
    estado_der = 0;
    estado_izq = 0;
    estado_arriba = 0;
    estado_abajo = 0;
    estado_esquina = 4;
    regresar = false;
}
```

Con esta función realizamos varios procesos, especialmente el movimiento y el proceso de llenado de nuestro arreglo para el
entrenamiento de la neurona
```
function update() { 

    if (!key_der.isDown && !key_izq.isDown && !key_down.isDown && !key_up.isDown){
        jugador.position.x=230;
        jugador.position.y=200;
    }
    
    if (!regresar) {
         timer += 1; 
    }
    posicion_inical();

    estado_der = 0;
    estado_izq = 0;
    estado_arriba = 0;
    estado_abajo = 0;
    estado_esquina = 0;
    
    coordenada_X = bala.position.x - jugador.position.x;  // Diferencia de las coordenadas para detectar el cuadrante
    coordenada_Y = bala.position.y - jugador.position.y;  // Diferencia de las coordenadas para detectar el cuadrante 
    dis_bala_jugador = Math.sqrt(coordenada_X * coordenada_X + coordenada_Y * coordenada_Y); // Fórmula de distancia bala-jugador

    if(!modoAuto){
        if (key_der.isDown && key_up.isDown) {
            esquinaArribaDerecha();
        } else if (key_izq.isDown && key_up.isDown) {
            esquinaArribaIzquierda();
        } else if (key_izq.isDown && key_down.isDown) {
            esquinaAbajoIzquierda();
        } else if (key_der.isDown && key_down.isDown) {
            esquinaAbajoDerecha();
        } else {
            if (key_der.isDown) {
                derecha();
            } else if (key_izq.isDown) {
                izquierda();
            } else if (key_up.isDown) {
                arriba();
            } else if (key_down.isDown) {
                abajo();
            }
        }
    }

    if (modoAuto && dis_bala_jugador <= 200) {
        var decision = datosDeEntrenamiento([dis_bala_jugador, coordenada_X, coordenada_Y]);
        console.log(decision);
        if (decision < 4) {
            // Si la decisión no es ir a una esquina, moverse en la dirección determinada por la red neuronal
            if (decision === 0) arriba();
            else if (decision === 1) abajo();
            else if (decision === 2) derecha();
            else if (decision === 3) izquierda();
        } else {
            // Si la decisión es ir a una esquina, determinar la esquina más alejada de la pelota
            var esquinaAlejada;
            if (bala.position.x <= w / 2) {
                // Si la pelota está a la izquierda del centro, la esquina más alejada está a la derecha
                if (bala.position.y <= h / 2) {
                    // Si la pelota está arriba del centro, la esquina más alejada es la esquina inferior derecha
                    esquinaAlejada = esquinaAbajoDerecha;
                } else {
                    // Si la pelota está abajo del centro, la esquina más alejada es la esquina superior derecha
                    esquinaAlejada = esquinaArribaDerecha;
                }
            } else {
                // Si la pelota está a la derecha del centro, la esquina más alejada está a la izquierda
                if (bala.position.y <= h / 2) {
                    // Si la pelota está arriba del centro, la esquina más alejada es la esquina inferior izquierda
                    esquinaAlejada = esquinaAbajoIzquierda;
                } else {
                    // Si la pelota está abajo del centro, la esquina más alejada es la esquina superior izquierda
                    esquinaAlejada = esquinaArribaIzquierda;
                }
            }
    
            // Seleccionar una esquina aleatoria diferente a la esquina más alejada
            var esquinasDisponibles = [esquinaArribaIzquierda, esquinaArribaDerecha, esquinaAbajoIzquierda, esquinaAbajoDerecha];
            esquinasDisponibles = esquinasDisponibles.filter(function(esquina) {
                return esquina !== esquinaAlejada;
            });
            var esquinaAleatoria = esquinasDisponibles[Math.floor(Math.random() * esquinasDisponibles.length)];
    
            // Mover al jugador a la esquina aleatoria seleccionada
            esquinaAleatoria();
        }
    }    

    juego.physics.arcade.collide(bala, jugador, colisionH, null, this);
    

    if (modoAuto == false) {
        var nuevoDatoEntrenamiento = {
            'input': [dis_bala_jugador, coordenada_X, coordenada_Y],
            'output': [estado_arriba, estado_abajo, estado_der, estado_izq, estado_esquina]
        };
        datosEntrenamiento.push(nuevoDatoEntrenamiento);
    }
}
```

Con esta función mandamos al personaje a la posición inicial en cada reinicio del juego o despues de cada colisión
```
function posicion_inical(){
    if (timer > 25) {
        jugador.position.x = Phaser.Math.linear(jugador.position.x, 230, 0.5); 
        jugador.position.y = Phaser.Math.linear(jugador.position.y, 200, 0.5);
        if (Math.abs(jugador.position.x - 230) < 1 && Math.abs(jugador.position.y - 200) < 1) {
            regresar = true;
            timer = 0;
            jugador.position.x = 230;
            jugador.position.y = 200;
        }
    }
}
```

Con esta función mandamos la pausa despues de cada colisión
```
function colisionH(){  
    pausa();     
}
```

Esta función es para renderizar codigo HTML y agregar información o detalles extras
```
function render(){}
```