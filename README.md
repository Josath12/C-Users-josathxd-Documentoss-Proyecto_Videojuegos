# C-Users-josathxd-Documentoss-Proyecto_Videojuegos
GestorVideojuegos
import subprocess
import os
import json
import requests
from dataclasses import dataclass, asdict, field

GENEROS = ["terror", "accion", "aventura", "deportes", "estrategia", "simulacion"]
PLATAFORMAS = ["PC", "PS4", "PS5", "XBOX ONE", "XBOX SERIES X", "NINTENDO SWITCH"]
CALIFICACIONES = ["E", "E10+", "T", "M", "AO"]
ARCHIVO_DATOS = "videojuegos.json"


def limpiar():
    if os.name == 'nt':
        subprocess.run('cls', shell=True)
    else:
        subprocess.run('clear', shell=True)


@dataclass
class Videojuego:
    nombre: str
    genero: str
    plataforma: str
    año: int
    calificacion: str
    estado: str
    id: str


class GestorVideojuegos:

    def __init__(self):
        self.DatosJuegos = []
        self.cargar_datos()

   
    # =========================================================
    def _pedir_opcion(self, mensaje, opciones):
        while True:
            valor = input(mensaje).strip()
            # Comparación insensible a mayúsculas/minúsculas (NUEVO)
            for opcion in opciones:
                if valor.lower() == opcion.lower():
                    return opcion
            limpiar()
            print("Opción inválida. Elige una de estas:")
            for opcion in opciones:
                print(" -", opcion)


    # =========================================================
    def _pedir_texto_obligatorio(self, mensaje):
        while True:
            valor = input(mensaje).strip()
            if valor != "":
                return valor
            print("Este campo no puede estar vacío.")


    # =========================================================
    def _pedir_año(self, mensaje):
        while True:
            valor = input(mensaje).strip()
            if valor.isdigit() and 1970 <= int(valor) <= 2100:
                return int(valor)
            print("Año inválido. Debe ser un número entre 1970 y 2100.")


    # =========================================================
    def _generar_id(self):
        if not self.DatosJuegos:
            return "1"
        return str(max(int(juego.id) for juego in self.DatosJuegos) + 1)

    def Agregar(self):
        # MODIFICADO: validamos que la cantidad ingresada sea un
        # número positivo, para evitar que el programa truene si
        # el usuario escribe texto o un número negativo.
        while True:
            cantidad_str = input("¿Cuántos videojuegos desea agregar? ")
            if cantidad_str.isdigit() and int(cantidad_str) > 0:
                cantidad = int(cantidad_str)
                break
            print("Ingresa un número entero positivo.")

        for _ in range(cantidad):
            nombre = self._pedir_texto_obligatorio("Ingrese el nombre del videojuego: ")

            limpiar()
            print("Géneros disponibles:", ", ".join(GENEROS))
            genero = self._pedir_opcion("Ingrese el género del videojuego: ", GENEROS)

            limpiar()
            print("Plataformas disponibles:", ", ".join(PLATAFORMAS))
            plataforma = self._pedir_opcion("Ingrese la plataforma del videojuego: ", PLATAFORMAS)

            año = self._pedir_año("Ingrese el año del videojuego: ")

            limpiar()
            print("Calificaciones disponibles:", ", ".join(CALIFICACIONES))
            calificacion = self._pedir_opcion("Ingrese la calificación del videojuego: ", CALIFICACIONES)

            estado = self._pedir_texto_obligatorio("Ingrese el estado del videojuego: ")

            nuevo_id = self._generar_id()
            juego = Videojuego(nombre, genero, plataforma, año, calificacion, estado, nuevo_id)
            self.DatosJuegos.append(juego)
            self.guardar_datos()
            print(f"✅ '{nombre}' agregado con id {nuevo_id}.")

    # =========================================================
    def Mostrar(self):
        if not self.DatosJuegos:
            print("No hay videojuegos registrados todavía.")
            return
        for juego in self.DatosJuegos:
            print("-" * 30)
            print(f"ID: {juego.id}")
            print(f"Nombre: {juego.nombre}")
            print(f"Género: {juego.genero}")
            print(f"Plataforma: {juego.plataforma}")
            print(f"Año: {juego.año}")
            print(f"Calificación: {juego.calificacion}")
            print(f"Estado: {juego.estado}")
        print("-" * 30)


    # =========================================================
    def buscar(self):
        nombre = input("Ingrese el nombre (o parte del nombre) a buscar: ").strip().lower()
        encontrados = [j for j in self.DatosJuegos if nombre in j.nombre.lower()]

        if not encontrados:
            print("No se encontró ningún videojuego con ese nombre.")
            return

        for juego in encontrados:
            print("-" * 30)
            print(f"ID: {juego.id}")
            print(f"Nombre: {juego.nombre}")
            print(f"Género: {juego.genero}")
            print(f"Plataforma: {juego.plataforma}")
            print(f"Año: {juego.año}")
            print(f"Calificación: {juego.calificacion}")
            print(f"Estado: {juego.estado}")
        print("-" * 30)


    # =========================================================
    def editar(self):
        if not self.DatosJuegos:
            print("No hay videojuegos para editar.")
            return

        id_editar = input("Ingrese el ID del videojuego a editar: ").strip()
        juego = next((j for j in self.DatosJuegos if j.id == id_editar), None)

        if juego is None:
            print("No se encontró ningún videojuego con ese ID.")
            return

        print("Deja el campo vacío para mantener el valor actual.")

        nuevo_nombre = input(f"Nombre [{juego.nombre}]: ").strip()
        if nuevo_nombre:
            juego.nombre = nuevo_nombre

        nuevo_genero = input(f"Género [{juego.genero}] (opciones: {', '.join(GENEROS)}): ").strip()
        if nuevo_genero:
            if nuevo_genero.lower() in [g.lower() for g in GENEROS]:
                juego.genero = nuevo_genero.lower()
            else:
                print("Género inválido, se mantuvo el valor anterior.")

        nueva_plataforma = input(f"Plataforma [{juego.plataforma}] (opciones: {', '.join(PLATAFORMAS)}): ").strip()
        if nueva_plataforma:
            match = next((p for p in PLATAFORMAS if p.lower() == nueva_plataforma.lower()), None)
            if match:
                juego.plataforma = match
            else:
                print("Plataforma inválida, se mantuvo el valor anterior.")

        nuevo_año = input(f"Año [{juego.año}]: ").strip()
        if nuevo_año:
            if nuevo_año.isdigit() and 1970 <= int(nuevo_año) <= 2100:
                juego.año = int(nuevo_año)
            else:
                print("Año inválido, se mantuvo el valor anterior.")

        nueva_calificacion = input(f"Calificación [{juego.calificacion}] (opciones: {', '.join(CALIFICACIONES)}): ").strip()
        if nueva_calificacion:
            match = next((c for c in CALIFICACIONES if c.lower() == nueva_calificacion.lower()), None)
            if match:
                juego.calificacion = match
            else:
                print("Calificación inválida, se mantuvo el valor anterior.")

        nuevo_estado = input(f"Estado [{juego.estado}]: ").strip()
        if nuevo_estado:
            juego.estado = nuevo_estado

        self.guardar_datos()
        print(f"✅ Videojuego '{juego.nombre}' actualizado correctamente.")


    # =========================================================
    def eliminar(self):
        if not self.DatosJuegos:
            print("No hay videojuegos para eliminar.")
            return

        print("IDs disponibles:", ", ".join(j.id for j in self.DatosJuegos))
        id_eliminar = input("Ingrese el ID del videojuego a eliminar: ").strip()

        juego = next((j for j in self.DatosJuegos if j.id == id_eliminar), None)
        if juego is None:
            print("No se encontró ningún videojuego con ese ID.")
            return

        confirmar = input(f"¿Seguro que quieres eliminar '{juego.nombre}'? (s/n): ").strip().lower()
        if confirmar == "s":
            self.DatosJuegos.remove(juego)
            self.guardar_datos()
            print(f"🗑️ Videojuego '{juego.nombre}' eliminado correctamente.")
        else:
            print("Eliminación cancelada.")


    # =========================================================
    def estadisticas(self):
        if not self.DatosJuegos:
            print("No hay videojuegos registrados todavía.")
            return

        print("1. Cantidad de videojuegos por género")
        print("2. Cantidad de videojuegos por plataforma")
        print("3. Cantidad de videojuegos por calificación")
        print("4. Cantidad de videojuegos por estado")
        opcion = input("Ingrese una opción: ").strip()

        campo_por_opcion = {
            "1": "genero",
            "2": "plataforma",
            "3": "calificacion",
            "4": "estado",
        }

        campo = campo_por_opcion.get(opcion)
        if campo is None:
            print("Opción no válida.")
            return

        conteo = {}
        for juego in self.DatosJuegos:
            valor = getattr(juego, campo)
            conteo[valor] = conteo.get(valor, 0) + 1

        # NUEVO: orden descendente por cantidad
        for clave, cantidad in sorted(conteo.items(), key=lambda x: x[1], reverse=True):
            print(f"{clave}: {cantidad}")


    # =========================================================
    def guardar_datos(self):
        try:
            lista_juegos = [asdict(juego) for juego in self.DatosJuegos]
            with open(ARCHIVO_DATOS, "w", encoding="utf-8") as archivo:
                json.dump(lista_juegos, archivo, indent=4, ensure_ascii=False)
        except OSError as error:
            print(f"⚠ No se pudo guardar el archivo: {error}")


    # =========================================================
    def cargar_datos(self):
        try:
            try:
                with open(ARCHIVO_DATOS, "r", encoding="utf-8") as archivo:
                    lista_juegos = json.load(archivo)
            except UnicodeDecodeError:
                with open(ARCHIVO_DATOS, "r", encoding="cp1252") as archivo:
                    lista_juegos = json.load(archivo)
            self.DatosJuegos = [Videojuego(**juego) for juego in lista_juegos]
        except FileNotFoundError:
            print("💾 No se encontró el archivo JSON. Se creará uno nuevo al guardar datos.")
            self.DatosJuegos = []
        except json.JSONDecodeError:
            print("⚠ El archivo JSON está dañado o vacío. Se iniciará con una lista vacía.")
            self.DatosJuegos = []
def enviar_api(self):
    url = "https://reqres.in"
    headers = {"Content-Type": "application/json"}
    lista_juegos = [asdict(juego) for juego in self.DatosJuegos]

    try:
        response = requests.post(url, json=lista_juegos, headers=headers)
        if response.status_code == 200:
            print("✅ Datos enviados correctamente a la API.")
        else:
            print(f"⚠ Error al enviar datos a la API: {response.status_code} - {response.text}")
    except requests.RequestException as e:
        print(f"⚠ Ocurrió un error al intentar enviar los datos: {e}")

def main():
    gestor = GestorVideojuegos()
    while True:
        print("\n" + "=" * 30)
        print("      GESTOR DE VIDEOJUEGOS      ")
        print("=" * 30)
        print("1.  Agregar Videojuegos")
        print("2.  Mostrar Catálogo")
        print("3.  Buscar Videojuego")
        print("4.  Editar Videojuego")   
        print("5.  Eliminar Videojuego")
        print("6.  Ver Estadísticas")
        print("7.  Enviar Datos a la API")
        print("8.  Salir")
        print("=" * 30)

        opcion = input("Selecciona una opción (1-8): ").strip()
        try:
            if opcion == "1":
                gestor.Agregar()
            elif opcion == "2":
                gestor.Mostrar()
            elif opcion == "3":
                gestor.buscar()
            elif opcion == "4":
                gestor.editar()
            elif opcion == "5":
                gestor.eliminar()
            elif opcion == "6":
                gestor.estadisticas()
            elif opcion == "7":
                gestor.enviar_api() # type: ignore
            elif opcion == "8":
                print("\n👋 ¡Gracias por usar el Gestor de Videojuegos! Hasta luego.")
                break
            else:
                print("\n⚠ Opción no válida. Por favor, intenta de nuevo.")
        except KeyboardInterrupt:
            print("\n\nOperación cancelada por el usuario.")
        except Exception as error:
            print(f"\n⚠ Ocurrió un error inesperado: {error}")


if __name__ == "__main__":
