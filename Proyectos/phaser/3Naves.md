```
// Definición del ancho y alto del lienzo del juego
var w = 800;
var h = 400;
var limiteMovimiento = 80; // Límite para el movimiento hacia adelante

// Declaración de variables para los elementos del juego
var jugador; // Jugador principal
var fondo; // Fondo del juego

var bala, balaD = false, nave; // Bala, indicador de si está en movimiento, y nave asociada
var bala2, balaD2 = false, nave2; // Segunda bala, indicador de movimiento y nave asociada
var bala3, balaD3 = false, nave3; // Tercera bala, indicador de movimiento y nave asociada

// Declaración de variables para las teclas de control
var salto; // Tecla para saltar
var menu; // Menú de pausa

// Variables para la velocidad y posición de las balas
var velocidadBala, velocidadBala2, velocidadBala3; // Velocidades de las balas
var despBala, despBala2, despBala3; // Distancias entre el jugador y las balas

// Variables para el estado del jugador
var estatusAire; // Indicador de si el jugador está en el aire
var estatuSuelo; // Indicador de si el jugador está en el suelo

// Variables para la red neuronal y el entrenamiento automático
var nnNetwork, nnEntrenamiento, nnSalida, datosEntrenamiento = []; // Variables relacionadas con la red neuronal
var modoAuto = false; // Indicador del modo de juego (manual o automático)
var eCompleto = false; // Indicador de si el entrenamiento de la red neuronal está completo
```
```
// Creación del juego con Phaser
var juego = new Phaser.Game(w, h, Phaser.CANVAS, '', { 
    preload: preload, 
    create: create, 
    update: update, 
    render: render 
});
```
```
// Función para precargar los recursos del juego
function preload() {
    juego.load.image('fondo', 'assets/game/fondo.jpg'); // Carga del fondo del juego
    juego.load.spritesheet('mono', 'assets/sprites/altair.png', 32, 48); // Carga del sprite del jugador
    juego.load.image('nave', 'assets/game/ufo.png'); // Carga del sprite de la nave enemiga
    juego.load.image('bala', 'assets/sprites/purple_ball.png'); // Carga del sprite de la bala
    juego.load.image('menu', 'assets/game/menu.png'); // Carga del sprite del menú de pausa
}
```
```
// Función para configurar los elementos al iniciar el juego
function create() {
    juego.physics.startSystem(Phaser.Physics.ARCADE); // Inicia el sistema de física del juego
    juego.physics.arcade.gravity.y = 800; // Establece la gravedad del juego

    fondo = juego.add.tileSprite(0, 0, w, h, 'fondo'); // Añade el fondo al juego como un tileSprite
    nave = juego.add.sprite(w - 100, h - 70, 'nave'); // Añade la nave enemiga al juego
    bala = juego.add.sprite(w - 100, h, 'bala'); // Añade la primera bala al juego
    nave2 = juego.add.sprite(w - 100, 0, 'nave'); // Añade otra nave enemiga al juego
    bala2 = juego.add.sprite(w - 100, 30, 'bala'); // Añade la segunda bala al juego
    nave3 = juego.add.sprite(50, 0, 'nave'); // Añade otra nave enemiga al juego
    bala3 = juego.add.sprite(50, 30, 'bala'); // Añade la tercera bala al juego
    jugador = juego.add.sprite(50, h, 'mono'); // Añade al jugador principal al juego

    juego.physics.enable([jugador, bala, bala2, bala3]); // Habilita la física para los elementos del juego
    jugador.body.collideWorldBounds = true; // Permite que el jugador colisione con los bordes del mundo
    jugador.animations.add('corre', [8, 9, 10, 11]); // Agrega la animación de correr al jugador
    jugador.animations.play('corre', 10, true); // Reproduce la animación de correr del jugador

    [bala, bala2, bala3].forEach(sprite => {
        sprite.body.collideWorldBounds = true; // Permite que las balas colisionen con los bordes del mundo
    });

    pausaL = juego.add.text(w - 100, 20, 'Pausa', { font: '20px Arial', fill: '#fff' }); // Añade el texto de pausa al juego
    pausaL.inputEnabled = true; // Habilita la entrada de clics en el texto de pausa
    pausaL.events.onInputUp.add(pausa, self); // Asigna la función de pausa al hacer clic en el texto
    juego.input.onDown.add(mPausa, self); // Asigna la función de pausa al hacer clic en cualquier parte del juego

    salto = juego.input.keyboard.addKey(Phaser.Keyboard.SPACEBAR); // Asigna la tecla de salto

    nnNetwork = new synaptic.Architect.Perceptron(6, 6, 6, 2); // Inicializa la red neuronal
    nnEntrenamiento = new synaptic.Trainer(nnNetwork); // Inicializa el entrenador de la red neuronal

    var cursors = juego.input.keyboard.createCursorKeys(); // Crea las teclas de flechas

    // Asigna eventos a las teclas de flechas (Entrada)
    cursors.left.onDown.add(moverIzquierda, this); // Asigna la función de mover a la izquierda al hacer clic en la flecha izquierda
    cursors.right.onDown.add(moverDerecha, this); // Asigna la función de mover a la derecha al hacer clic en la flecha derecha
    cursors.up.onDown.add(saltar, this); // Asigna la función de saltar al hacer clic en la flecha arriba
}
```
```
// Función para mover al jugador a la izquierda
function moverIzquierda() {
    jugador.body.velocity.x = 0; // Ajusta la velocidad del jugador hacia la izquierda
}
```
```
// Función para mover al jugador a la derecha
function moverDerecha() {
    console.log(limiteMovimiento);
    if (jugador.position.x < limiteMovimiento) {
        jugador.body.velocity.x = 150; // Ajusta la velocidad del jugador hacia la derecha
    } else {
        jugador.body.velocity.x = 0; // Detiene el movimiento si supera el límite
    }
}
```
```
// Función para entrenar la red neuronal
function enRedNeural() {
    nnEntrenamiento.train(datosEntrenamiento, { rate: 0.0003, iterations: 10000, shuffle: true }); // Entrena la red neuronal con los datos de entrenamiento
}
```
```
// Función para obtener la salida de la red neuronal
function datosDeEntrenamiento(param_entrada) {
    nnSalida = nnNetwork.activate(param_entrada); // Activa la red neuronal con la entrada proporcionada
    return nnSalida[0] >= nnSalida[1]; // Devuelve verdadero si la salida indica saltar
}
```
```
// Función para pausar el juego
function pausa() {
    juego.paused = true; // Pausa el juego
    menu = juego.add.sprite(w / 2, h / 2, 'menu'); // Añade el menú de pausa al juego
    menu.anchor.setTo(0.5, 0.5); // Establece el punto de anclaje del menú de pausa
}
```
```
// Función para gestionar la pausa del juego
function mPausa(event) {
    // Verifica si el juego está pausado
    if (juego.paused) {
        // Calcula las coordenadas del menú de pausa
        var menu_x1 = w / 2 - 270 / 2,
            menu_x2 = w / 2 + 270 / 2,
            menu_y1 = h / 2 - 180 / 2,
            menu_y2 = h / 2 + 180 / 2;

        // Obtiene las coordenadas del mouse al hacer clic (Entrada)
        var mouse_x = event.x,
            mouse_y = event.y;

        // Verifica si el clic se realizó dentro del área del menú de pausa
        if (mouse_x > menu_x1 && mouse_x < menu_x2 && mouse_y > menu_y1 && mouse_y < menu_y2) {
            // Verifica si se hizo clic en la opción de reiniciar el entrenamiento
            if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 && mouse_y <= menu_y1 + 90) {
                // Reinicia las variables relacionadas con el entrenamiento
                eCompleto = false;
                datosEntrenamiento = [];
                modoAuto = false;
            } else if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 + 90 && mouse_y <= menu_y2) {
                // Verifica si se hizo clic en la opción de activar el modo automático
                if (!eCompleto) {
                    enRedNeural(); // Entrena la red neuronal si no está completo el proceso de entrenamiento
                    eCompleto = true; // Marca el proceso de entrenamiento como completo
                }
                modoAuto = true; // Activa el modo automático
            }

            menu.destroy(); // Elimina el menú de pausa
            resetVariables(); // Reinicia las variables del juego
            resetVariables2(); // Reinicia otras variables relacionadas
            resetVariables3(); // Reinicia otras variables relacionadas
            juego.paused = false; // Desactiva la pausa del juego
        }
    }
}
```
```
// Función para reiniciar las variables del juego
function resetVariables() {
    jugador.body.velocity.x = 0; // Reinicia la velocidad horizontal del jugador
    jugador.body.velocity.y = 0; // Reinicia la velocidad vertical del jugador
    bala.body.velocity.x = 0; // Reinicia la velocidad horizontal de la primera bala
    bala.position.x = w - 100; // Reinicia la posición horizontal de la primera bala
    jugador.position.x = 50; // Reinicia la posición horizontal del jugador
    balaD = false; // Restaura el estado de disparo de la primera bala

    bala2.body.velocity.y = 0; // Reinicia la velocidad vertical de la segunda bala
    bala2.position.x = w - 100; // Reinicia la posición horizontal de la segunda bala
    bala2.position.y = 30; // Reinicia la posición vertical de la segunda bala
    balaD2 = false; // Restaura el estado de disparo de la segunda bala

    bala3.body.velocity.y = 0; // Reinicia la velocidad vertical de la tercera bala
    bala3.position.y = 50; // Reinicia la posición vertical de la tercera bala
    bala3.body.velocity.x = 0; // Reinicia la velocidad horizontal de la tercera bala
    bala3.position.x = 50; // Reinicia la posición horizontal de la tercera bala
    balaD3 = false; // Restaura el estado de disparo de la tercera bala
}
```
```
// Función para reiniciar variables relacionadas
function resetVariables2() {
    console.log("reinicio2"); // Imprime un mensaje indicando el reinicio de variables
}
```
```
// Función para reiniciar variables relacionadas
function resetVariables3() {
    console.log("reinicio3"); // Imprime un mensaje indicando el reinicio de variables
}
```
```
// Función para hacer que el jugador salte
function saltar() {
    jugador.body.velocity.y = -270; // Establece la velocidad vertical para que el jugador salte
}
```
```
// Función que se ejecuta en cada fotograma del juego para actualizar su estado
function update() {
    fondo.tilePosition.x -= 1; // Desplaza el fondo del juego

    // Detecta y gestiona las colisiones entre las balas y el jugador
    juego.physics.arcade.collide(bala, jugador, colisionH, null, this);
    juego.physics.arcade.collide(bala2, jugador, colisionH, null, this);
    juego.physics.arcade.collide(bala3, jugador, colisionH, null, this);

    estatuSuelo = 1; // Establece el estado del suelo a 1 por defecto
    estatusAire = 0; // Establece el estado del aire a 0 por defecto

    // Verifica si el jugador no está en el suelo
    if (!jugador.body.onFloor()) {
        estatuSuelo = 0; // Si no está en el suelo, establece el estado del suelo a 0
        estatusAire = 1; // Si no está en el suelo, establece el estado del aire a 1
    }

    // Calcula la distancia entre el jugador y las balas
    despBala = Math.floor(jugador.position.x - bala.position.x);
    despBala2 = Math.floor(jugador.position.x - bala2.position.x);
    despBala3 = Math.floor(jugador.position.y - bala3.position.y);

    // Verifica si el modo automático está desactivado y se presiona la tecla de salto y el jugador está en el suelo
    if (modoAuto == false && salto.isDown && jugador.body.onFloor()) {
        saltar(); // Hace que el jugador salte
    }

    // Verifica si el modo automático está activado y la primera bala está en el aire
    if (modoAuto == true && bala.position.x > 0 && jugador.body.onFloor()) {
        // Verifica si la salida de la red neuronal indica que debe saltar
        if (datosDeEntrenamiento([despBala, velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3])) {
            saltar(); // Hace que el jugador salte
            // Ejemplo: si el jugador está a la derecha del centro, muévelo hacia la izquierda; de lo contrario, muévelo hacia la derecha
            if (jugador.position.x > 400) {
                moverIzquierdaAuto();
            } else {
                moverDerechaAuto();
            }
        }
    }

    // Verifica si la primera bala no ha sido disparada
    if (balaD == false) {
        disparo(); // Dispara la primera bala
    }

    // Verifica si la posición vertical de la segunda bala es menor o igual a cero
    if (bala2.position.y <= 0) {
        resetVariables2(); // Reinicia las variables relacionadas con la segunda bala
    }

    // Verifica si la segunda bala no ha sido disparada
    if (balaD2 == false) {
        disparo2(); // Dispara la segunda bala
    }

    // Verifica si la posición vertical de la tercera bala es mayor o igual a la altura del juego
    if (bala3.position.y >= h) {
        resetVariables3(); // Reinicia las variables relacionadas con la tercera bala
    }

    // Verifica si la tercera bala no ha sido disparada
    if (balaD3 == false) {
        disparo3(); // Dispara la tercera bala
    }

    // Verifica si la primera bala ha salido del área del juego
    if (bala.position.x <= 0) {
        resetVariables(); // Reinicia las variables relacionadas con la primera bala
    }

    // Limita el movimiento del jugador si supera el límite de movimiento
    if (jugador.position.x > limiteMovimiento) {
        jugador.position.x = limiteMovimiento;
    }

    // Verifica si el modo automático está desactivado y la primera bala ha sido disparada
    if (modoAuto == false && bala.position.x > 0) {
        // Registra los datos de entrenamiento para la red neuronal
        datosEntrenamiento.push({
            'input': [despBala, velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3],
            'output': [estatusAire, estatuSuelo, estatusAire, estatuSuelo]
        });
    }
}
```
```
// Función para disparar la primera bala
function disparo() {
    velocidadBala = -1 * 400; // Establece la velocidad horizontal de la bala
    bala.body.velocity.y = 0; // Detiene el movimiento vertical de la bala
    bala.body.velocity.x = velocidadBala; // Establece la velocidad horizontal de la bala
    balaD = true; // Marca la bala como disparada
}

// Función para disparar la segunda bala
function disparo2() {
    velocidadBala2 = -1 * 800; // Establece la velocidad horizontal de la segunda bala
    bala2.body.velocity.y = 0; // Detiene el movimiento vertical de la segunda bala
    bala2.body.velocity.x = velocidadBala2; // Establece la velocidad horizontal de la segunda bala
    bala2.body.gravity.y = 1000; // Aplica gravedad a la segunda bala
    balaD2 = true; // Marca la segunda bala como disparada
}

// Función para disparar la tercera bala
function disparo3() {
    velocidadBala3 = 300; // Establece la velocidad vertical de la tercera bala
    bala3.body.velocity.y = velocidadBala3; // Establece la velocidad vertical de la tercera bala
    bala3.body.velocity.x = 0; // Detiene el movimiento horizontal de la tercera bala
    balaD3 = true; // Marca la tercera bala como disparada
}
```
```
// Función para mover automáticamente hacia la izquierda
function moverIzquierdaAuto() {
    jugador.body.velocity.x = -150; // Ajusta la velocidad del jugador hacia la izquierda
}

// Función para mover automáticamente hacia la derecha
function moverDerechaAuto() {
    jugador.body.velocity.x = 150; // Ajusta la velocidad del jugador hacia la derecha
}
```
```
// Función que se llama cuando hay una colisión horizontal
function colisionH() {
    pausa(); // Pausa el juego
}
```
```
// Función de renderizado
function render() {}
```
// Entradas:
// Teclas, click, posiciones, velocidades, pausa, modo de entrenamiento

// Salidas:
// Movimiento del jugador, salto, disparos, colisiones, actualización de variables de entrenamiento para la red neuronal