"""
JUEGO: CIVILES E IMPOSTORES
Versión de consola en Python 3.10+
Ejecutar con: python juego_civiles_impostores.py

Reglas:
- Se necesitan mínimo 3 jugadores.
- Se elige una palabra secreta al azar.
- Los civiles conocen la palabra; los impostores solo conocen la categoría.
- Cada jugador vivo da una pista por ronda.
- Después de las pistas, todos votan para eliminar a alguien.
- Los civiles ganan si eliminan a todos los impostores.
- Los impostores ganan si igualan o superan en número a los civiles.
"""

from dataclasses import dataclass
from enum import Enum
import random


class Rol(Enum):
    CIVIL = "Civil"
    IMPOSTOR = "Impostor"


@dataclass
class Jugador:
    nombre: str
    rol: Rol
    activo: bool = True


@dataclass(frozen=True)
class PalabraSecreta:
    palabra: str
    categoria: str


@dataclass
class Pista:
    jugador: str
    texto: str


@dataclass
class Voto:
    votante: str
    votado: str


class JuegoCivilesImpostores:
    def __init__(self):
        self.jugadores = []
        self.palabra_secreta = None
        self.numero_ronda = 0
        self.pistas = []
        self.votos = []

        self.banco_palabras = [
            PalabraSecreta("Elefante", "Animales"),
            PalabraSecreta("Delfín", "Animales"),
            PalabraSecreta("Tigre", "Animales"),
            PalabraSecreta("Hospital", "Lugares"),
            PalabraSecreta("Aeropuerto", "Lugares"),
            PalabraSecreta("Biblioteca", "Lugares"),
            PalabraSecreta("Guitarra", "Objetos"),
            PalabraSecreta("Paraguas", "Objetos"),
            PalabraSecreta("Computador", "Tecnología"),
            PalabraSecreta("Celular", "Tecnología"),
            PalabraSecreta("Baloncesto", "Deportes"),
            PalabraSecreta("Fútbol", "Deportes"),
            PalabraSecreta("Pizza", "Comidas"),
            PalabraSecreta("Arepa", "Comidas"),
            PalabraSecreta("Montaña", "Naturaleza"),
            PalabraSecreta("Volcán", "Naturaleza"),
        ]

    def configurar_partida(self):
        print("\n========== CIVILES E IMPOSTORES ==========")
        while True:
            try:
                cantidad = int(input("¿Cuántos jugadores van a participar? "))
                if cantidad < 3:
                    print("Deben participar mínimo 3 jugadores.")
                    continue
                break
            except ValueError:
                print("Escribe un número válido.")

        nombres = []
        for i in range(cantidad):
            while True:
                nombre = input(f"Nombre del jugador {i + 1}: ").strip()
                if not nombre:
                    print("El nombre no puede estar vacío.")
                elif nombre.casefold() in [n.casefold() for n in nombres]:
                    print("Ese nombre ya está registrado.")
                else:
                    nombres.append(nombre)
                    break

        while True:
            try:
                maximo = cantidad - 1
                impostores = int(input(f"¿Cuántos impostores? (1 a {maximo}): "))
                if 1 <= impostores <= maximo:
                    break
                print(f"Elige un número entre 1 y {maximo}.")
            except ValueError:
                print("Escribe un número válido.")

        palabra = random.choice(self.banco_palabras)
        indices_impostores = set(random.sample(range(cantidad), impostores))
        self.jugadores = [
            Jugador(nombre, Rol.IMPOSTOR if i in indices_impostores else Rol.CIVIL)
            for i, nombre in enumerate(nombres)
        ]
        self.palabra_secreta = palabra

    def mostrar_roles_en_privado(self):
        print("\n========== REPARTO DE ROLES ==========")
        print("Pasa el dispositivo a cada jugador. Nadie más debe mirar la pantalla.")
        input("\nPresiona ENTER para comenzar el reparto...")
        for jugador in self.jugadores:
            print("\n" * 35)
            print(f"Turno de: {jugador.nombre}")
            input("Solo esta persona debe presionar ENTER para ver su rol...")
            print("\n" * 4)
            print(f"TU ROL: {jugador.rol.value.upper()}")
            if jugador.rol == Rol.CIVIL:
                print(f"PALABRA SECRETA: {self.palabra_secreta.palabra.upper()}")
            else:
                print(f"CATEGORÍA: {self.palabra_secreta.categoria.upper()}")
                print("Debes descubrir la palabra sin que te descubran.")
            input("\nMemoriza tu información. Presiona ENTER y devuelve el dispositivo.")
        print("\n" * 35)
        print("¡Todos conocen su rol! Comienza la partida.")

    def jugadores_activos(self):
        return [j for j in self.jugadores if j.activo]

    def buscar_jugador(self, nombre):
        for jugador in self.jugadores:
            if jugador.nombre.casefold() == nombre.casefold() and jugador.activo:
                return jugador
        return None

    def mostrar_estado(self):
        activos = self.jugadores_activos()
        print("\nJugadores que siguen en juego:")
        for jugador in activos:
            print(f" - {jugador.nombre}")
        print(f"Jugadores activos: {len(activos)}")

    def recoger_pistas(self):
        self.pistas = []
        print("\n========== RONDA DE PISTAS ==========")
        print("Cada jugador activo debe decir una pista relacionada con la palabra.")
        print("No digan la palabra secreta directamente.")
        for jugador in self.jugadores_activos():
            print(f"\nTurno de {jugador.nombre}.")
            while True:
                pista = input("Escribe la pista que dijo el jugador: ").strip()
                if not pista:
                    print("La pista no puede estar vacía.")
                elif len(pista) > 80:
                    print("La pista no puede superar 80 caracteres.")
                else:
                    self.pistas.append(Pista(jugador.nombre, pista))
                    break
        print("\nPistas de esta ronda:")
        for pista in self.pistas:
            print(f"{pista.jugador}: {pista.texto}")

    def realizar_votacion(self):
        self.votos = []
        activos = self.jugadores_activos()
        print("\n========== VOTACIÓN ==========")
        print("Cada jugador activo debe votar por otra persona.")
        for votante in activos:
            while True:
                print(f"\nVota {votante.nombre}.")
                print("Candidatos: " + ", ".join(j.nombre for j in activos if j.nombre != votante.nombre))
                nombre_votado = input("Nombre de la persona que quieres eliminar: ").strip()
                votado = self.buscar_jugador(nombre_votado)
                if votado is None:
                    print("Ese jugador no existe o ya fue eliminado.")
                elif votado.nombre == votante.nombre:
                    print("No puedes votarte a ti mismo.")
                else:
                    self.votos.append(Voto(votante.nombre, votado.nombre))
                    break

        conteo = {}
        for voto in self.votos:
            conteo[voto.votado] = conteo.get(voto.votado, 0) + 1

        print("\nResultados de la votación:")
        for nombre, cantidad in sorted(conteo.items()):
            print(f"{nombre}: {cantidad} voto(s)")

        mayor = max(conteo.values())
        empatados = [nombre for nombre, cantidad in conteo.items() if cantidad == mayor]
        if len(empatados) > 1:
            print("\n¡Hay empate! Nadie es eliminado en esta ronda.")
            return

        eliminado = self.buscar_jugador(empatados[0])
        eliminado.activo = False
        print(f"\n{eliminado.nombre} ha sido eliminado.")
        print(f"Su rol era: {eliminado.rol.value.upper()}")

    def verificar_ganador(self):
        activos = self.jugadores_activos()
        civiles = sum(1 for j in activos if j.rol == Rol.CIVIL)
        impostores = sum(1 for j in activos if j.rol == Rol.IMPOSTOR)

        if impostores == 0:
            return "CIVILES"
        if impostores >= civiles:
            return "IMPOSTORES"
        return None

    def mostrar_resultado_final(self, ganador):
        print("\n" * 2)
        print("========== FIN DE LA PARTIDA ==========")
        print(f"¡GANARON LOS {ganador}!")
        print(f"La palabra secreta era: {self.palabra_secreta.palabra}")
        print("\nRoles finales:")
        for jugador in self.jugadores:
            estado = "En juego" if jugador.activo else "Eliminado"
            print(f"- {jugador.nombre}: {jugador.rol.value} ({estado})")

    def jugar(self):
        self.configurar_partida()
        self.mostrar_roles_en_privado()

        ganador = self.verificar_ganador()
        while ganador is None:
            self.numero_ronda += 1
            print(f"\n\n========== RONDA {self.numero_ronda} ==========")
            self.mostrar_estado()
            self.recoger_pistas()
            input("\nTiempo de discusión. Cuando estén listos, presiona ENTER para votar...")
            self.realizar_votacion()
            ganador = self.verificar_ganador()

        self.mostrar_resultado_final(ganador)


def main():
    while True:
        juego = JuegoCivilesImpostores()
        juego.jugar()
        otra = input("\n¿Quieren jugar otra partida? (s/n): ").strip().lower()
        if otra not in ("s", "si", "sí"):
            print("¡Gracias por jugar!")
            break


if __name__ == "__main__":
    main()
