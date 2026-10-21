# Practica_VR_01
Practica 01 de la asignatura Realidad Virtual

Simulador de Tiro VR - Unreal Engine 5

Práctica universitaria de Realidad Virtual desarrollada en Unreal Engine 5 (Blueprints). Es una galería de tiro arcade, pero el foco real del proyecto está en la arquitectura del código: aplicación estricta de Programación Orientada a Objetos (POO), optimización y buenas prácticas.

Qué incluye el juego

4 armas distintas:

Pistola: Recarga física acercando el cargador a la empuñadura y amartillado manual pasando la mano libre por encima.

Arco: Cálculo de tensión basado en la distancia entre las manos al soltar (sin físicas inestables de cuerda).

Hacha arrojadiza: Hereda la inercia del mando al soltarse y apaga sus físicas en el frame del impacto para quedarse clavada en las dianas.

Subfusil (Custom): Arma automática construida sobre la herencia de la pistola.

Modo Arcade: 10 rondas con límite de tiempo de 45 segundos. Hay que alcanzar un umbral de puntos que va subiendo en cada ronda. Al final de la ronda 10, ganas.

Sistema de dianas: Enemigos que avanzan de frente hacia el jugador (+100 puntos) y civiles que se cruzan por la línea de fuego para molestar (-200 puntos y penalización de tiempo).
