# 🧟 Momias de Guanajuato: La Leyenda

> **Desarrollo Móvil Integral — 10mo Cuatrimestre**  
> *Proyecto complementario de práctica TDD (Test-Driven Development)*  
> **Temática:** Plataformas 2D con física real y arquitectura desacoplada en Flutter.

---

## Descripción del Proyecto

**Momias de Guanajuato: La Leyenda** es un videojuego de plataformas en 2D desarrollado al 100% bajo la metodología **Test-Driven Development (TDD)**. El jugador encarna a un valiente explorador que debe recorrer los callejones empedrados de Guanajuato, sortear precipicios, saltar entre cornisas coloniales, derrotar o esquivar momias patrulleras y recolectar calaveritas de azúcar hasta alcanzar la meta legendaria: **El Callejón del Beso**.

A diferencia de juegos por turnos o movimientos discretos, este plataformas ejecuta un ciclo de física continuo (gravedad, inercia, salto, resolución de colisiones AABB e inteligencia artificial de patrulla). Toda la lógica central fue implementada en **Dart puro** dentro de `lib/logica/`, garantizando independencia absoluta de la interfaz gráfica y facilitando su cobertura completa mediante pruebas automatizadas antes de diseñar los widgets de Flutter.

---

## Mecánicas del Juego

* **Gravedad y Salto:** El explorador experimenta aceleración vertical continua por gravedad ($0.8\text{ px/frame}^2$) y aterrizaje firme sobre plataformas. El salto (impulso de $-12$) únicamente puede ejecutarse si el personaje se encuentra con los pies en el suelo.
* **Colisiones AABB (Axis-Aligned Bounding Box):** Detección precisa de traslape espacial bidimensional entre personaje, plataformas, calaveritas y momias.
* **Patrullaje Inteligente de Momias:** Enemigos que patrullan continuamente entre dos límites predeterminados invirtiendo su vector de velocidad horizontal al llegar a los extremos.
* **Sistema de Combate ("Stomp"):**
  * Si el jugador cae sobre la **mitad superior** de la momia ($v_y > 0$), la derrota al instante y obtiene un impulso de rebote.
  * Si choca contra la momia de lado o desde abajo, el jugador recibe daño y pierde una vida.
* **Coleccionables:** Recolección de calaveritas de azúcar que otorgan $+50$ puntos cada una al hacer contacto (solo una vez por calaverita).
* **Vidas y Game Over:** El jugador inicia con 3 vidas. Al recibir daño reaparece en la posición inicial; si las vidas caen a 0 se declara estado de derrota (`EstadoJuego.perdido`).
* **Condición de Victoria:** Cruzar el umbral del nivel (posición meta $x \ge 1900$) para alcanzar el Callejón del Beso (`EstadoJuego.ganado`).

---

## Estructura del Código y Entregables

```text
momias_guanajuato/
├── lib/
│   ├── logica/                      # Reglas de negocio y motor de física (Dart puro)
│   │   ├── rectangulo.dart          # Algoritmo de colisión de cajas delimitadoras (AABB)
│   │   ├── personaje.dart           # Entidad del jugador (posición, velocidad, caja)
│   │   ├── momia.dart               # Entidad enemiga con IA de patrullaje
│   │   ├── calaverita.dart          # Objeto coleccionable de puntaje
│   │   ├── motor_fisica.dart        # Gravedad, salto, colisión con suelo y límites
│   │   ├── motor_combate.dart       # Resolución de combate (stomp vs daño)
│   │   ├── estado_juego.dart        # Estados: jugando, ganado, perdido
│   │   └── motor_juego.dart         # Orquestador del ciclo de juego y reglas
│   ├── presentation/
│   │   └── juego_screen.dart        # Vista Flutter: cámara lateral, HUD y controles
│   └── main.dart                    # Punto de entrada de la aplicación
├── test/                            # Batería de pruebas unitarias e integración (TDD)
│   ├── rectangulo_test.dart         # Pruebas de intersección geométrica AABB
│   ├── motor_fisica_test.dart       # Pruebas de cinemática, salto y plataformas
│   ├── momia_test.dart              # Pruebas de patrullaje e inversión de límites
│   ├── motor_combate_test.dart      # Pruebas de combate y daño
│   └── motor_juego_test.dart        # Pruebas de vidas, puntaje, meta e integración
└── README.md                        # Documentación técnica y entregables del proyecto
```

---

## Rondas de Desarrollo TDD (Paso a Paso)

El proyecto fue construido siguiendo estrictamente el ciclo **Rojo 🔴 $\rightarrow$ Verde 🟢 $\rightarrow$ Refactorización 🔵**:

| Ronda | Componente | Comportamiento Probado / Implementado |
| :---: | :--- | :--- |
| **1** | `Rectangulo` | Algoritmo AABB: detección de traslape e intersección entre cajas delimitadoras en 2D. |
| **2** | `MotorFisica` + `Personaje` | Aceleración vertical por gravedad y restricción de salto exclusivo cuando el personaje pisa suelo. |
| **3** | `MotorFisica.resolverColisionConPiso` | Aterrizaje sobre plataformas usando parámetro-costura `yAnterior` para evitar traspaso en caída. |
| **4** | `MotorFisica.moverHorizontal` | Desplazamiento horizontal con restricción de bordes del nivel ($x \ge 0$ y $x + \text{ancho} \le \text{anchoNivel}$). |
| **5** | `Momia.patrullar` | Movimiento constante entre límites izquierdo/derecho e inversión de dirección determinista. |
| **6** | `MotorCombate.resolverColision` | Caída sobre la mitad superior elimina momia y otorga rebote; impacto lateral resta vida. |
| **7** | `MotorJuego.recolectarSiCorresponde` | Recolección de calaveritas, incremento de 50 puntos y bloqueo de recolección múltiple. |
| **8** | `MotorFisica.haLlegadoALaMeta` | Verificación de superación del umbral de posición del Callejón del Beso. |
| **9** | Vidas y Game Over en `MotorJuego` | Pérdida de vida, reaparición en coordenadas iniciales y cambio a `EstadoJuego.perdido` al llegar a 0. |
| **10** | Orquestación en `MotorJuego.mover` | Secuencia determinista: física $\rightarrow$ colisión plataformas $\rightarrow$ patrulla e impacto momias $\rightarrow$ calaveritas $\rightarrow$ meta. |
| **Int.** | Prueba de Integración | Verificación integral de movimiento, gravedad, colisión y victoria en un solo flujo. |

---

## Reporte de Pruebas Unitarias y Análisis

### Resultados de `flutter test`
Se cuenta con **23 pruebas unitarias e integración**, todas pasando con éxito en verde:

```text
00:00 +0: momia_test.dart: la momia avanza en linea recta mientras no llegue a un limite
00:00 +1: momia_test.dart: la momia invierte direccion al llegar al limite derecho
00:00 +2: momia_test.dart: una momia derrotada no se mueve
00:00 +3: motor_combate_test.dart: caer sobre la mitad superior de una momia la derrota
00:00 +4: motor_combate_test.dart: chocar de lado con una momia golpea al jugador
00:00 +5: motor_combate_test.dart: sin interseccion no pasa nada
00:01 +6: motor_fisica_test.dart: aplicarGravedad aumenta la velocidad vertical y mueve al personaje hacia abajo
00:01 +7: motor_fisica_test.dart: saltar solo funciona si el personaje esta en el suelo
00:01 +8: motor_fisica_test.dart: saltar no hace nada si el personaje ya esta en el aire
00:01 +9: motor_fisica_test.dart: el personaje aterriza sobre una plataforma al caer sobre ella
00:01 +10: motor_fisica_test.dart: el personaje no aterriza si todavia esta muy por encima de la plataforma
00:01 +11: motor_fisica_test.dart: moverHorizontal desplaza al personaje segun el delta recibido
00:01 +12: motor_fisica_test.dart: moverHorizontal no deja salir al personaje por el borde izquierdo
00:01 +13: motor_fisica_test.dart: moverHorizontal no deja salir al personaje por el borde derecho
00:01 +14: motor_fisica_test.dart: haLlegadoALaMeta es verdadero cuando el personaje alcanza la posicion
00:01 +15: motor_fisica_test.dart: haLlegadoALaMeta es falso si aun no llega
00:01 +16: motor_juego_test.dart: recolectar una calaverita suma puntos y la marca como recolectada
00:01 +17: motor_juego_test.dart: una calaverita ya recolectada no vuelve a sumar puntos
00:01 +18: motor_juego_test.dart: al chocar lateralmente con una momia se pierde una vida y el personaje regresa al inicio
00:01 +19: motor_juego_test.dart: perder la ultima vida cambia el estado a perdido
00:01 +20: motor_juego_test.dart: llegar a la meta cambia el estado a ganado
00:02 +21: rectangulo_test.dart: dos rectangulos que se traslapan se detectan como interseccion
00:02 +22: rectangulo_test.dart: dos rectangulos separados no se intersectan
00:03 +23: All tests passed!
```

### Resultados de `flutter analyze`
```text
Analyzing momias_guanajuato...
No issues found!
```

---

## Checklist de Cumplimiento TDD

| Estado | Verificación Requerida |
| :---: | :--- |
| **✓** | La colisión AABB (`Rectangulo`) se probó de forma aislada antes de usarse en cualquier otra parte. |
| **✓** | `resolverColisionConPiso` usa la posición anterior del personaje (`yAnterior`), distinguiendo caídas válidas de traspasos internos. |
| **✓** | El criterio de *stomp* exige caer ($v_y > 0$) Y hacer contacto en la mitad superior de la momia. |
| **✓** | `patrullar()` y la lógica sensible al tiempo son 100% deterministas en los tests. |
| **✓** | `MotorJuego.mover()` es el único orquestador central que define el orden de física, combate, recolección y meta. |
| **✓** | La interfaz gráfica (`JuegoScreen`) no toma decisiones de colisiones ni puntaje; únicamente mapea el estado del motor a píxeles. |

---

## Instrucciones de Ejecución

### Prerrequisitos
* Flutter SDK (3.16.x o superior)
* Dart SDK (3.2.x o superior)

### 1. Ejecutar la suite de pruebas unitarias
```bash
flutter test
```

### 2. Ejecutar el análisis de código
```bash
flutter analyze
```

### 3. Iniciar el juego en emulador o dispositivo
```bash
flutter run
```

### Controles
* **Botón Flecha Izquierda (⬅️):** Mantén presionado para desplazarte hacia la izquierda.
* **Botón Flecha Derecha (➡️):** Mantén presionado para desplazarte hacia la derecha.
* **Botón Salto (⬆️):** Presiona para brincar sobre plataformas o stompear momias.

---

## Reconocimiento Obtenido
> **Insignia Desbloqueada:** *Cinturón Negro de Proyecto* — Plataformas completo con física real, detección de colisiones, enemigos con IA y cámara de seguimiento, 100% guiado por pruebas TDD.

---

## Capturas de pantalla
