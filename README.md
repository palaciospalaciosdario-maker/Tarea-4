from abc import ABC, abstractmethod
import datetime

# =========================
# Validaciones
# =========================
def validar_correo(correo):
    return "@" in correo and "." in correo

# =========================
# Registro de logs
# =========================
def registrar_evento(mensaje):
    with open("logs.txt", "a") as f:
        f.write(f"{datetime.datetime.now()} - {mensaje}\n")

# =========================
# Clase abstracta Entidad
# =========================
class Entidad(ABC):
    @abstractmethod
    def mostrar_info(self):
        pass

# =========================
# Clase Cliente
# =========================
class Cliente(Entidad):
    def __init__(self, nombre, identificacion, correo):
        if not nombre or not identificacion or not correo:
            raise ValueError("Todos los campos son obligatorios")
        if not validar_correo(correo):
            raise ValueError("Correo inválido")
        
        self.__nombre = nombre
        self.__identificacion = identificacion
        self.__correo = correo

    def mostrar_info(self):
        return f"Cliente: {self.__nombre}, ID: {self.__identificacion}, Correo: {self.__correo}"

# =========================
# Clase abstracta Servicio
# =========================
class Servicio(ABC):
    @abstractmethod
    def calcular_costo(self, duracion):
        pass

    @abstractmethod
    def descripcion(self):
        pass

# =========================
# Servicios especializados
# =========================
class Sala(Servicio):
    def calcular_costo(self, duracion):
        return 50 * duracion  # costo por hora

    def descripcion(self):
        return "Reserva de sala de reuniones"

class Equipo(Servicio):
    def calcular_costo(self, duracion):
        return 30 * duracion  # costo por día

    def descripcion(self):
        return "Alquiler de equipos tecnológicos"

class Asesoria(Servicio):
    def calcular_costo(self, duracion):
        return 100 * duracion  # costo por sesión

    def descripcion(self):
        return "Asesoría especializada"

# =========================
# Clase Reserva
# =========================
class Reserva:
    def __init__(self, cliente, servicio, duracion):
        self.cliente = cliente
        self.servicio = servicio
        self.duracion = duracion
        self.estado = "Pendiente"

    def confirmar(self):
        try:
            costo = self.servicio.calcular_costo(self.duracion)
            self.estado = "Confirmada"
            return f"Reserva confirmada. Costo: {costo}"
        except Exception as e:
            registrar_evento(f"Error en confirmación: {str(e)}")
            self.estado = "Error"
            return "No se pudo confirmar la reserva."

    def cancelar(self):
        self.estado = "Cancelada"
        return "Reserva cancelada."

# =========================
# Menú principal
# =========================
def mostrar_menu():
    print("\n=== Software FJ ===")
    print("1. Registrar cliente")
    print("2. Crear servicio")
    print("3. Realizar reserva")
    print("4. Salir")

# =========================
# Programa principal
# =========================
def main():
    clientes = []
    servicios = {"1": Sala(), "2": Equipo(), "3": Asesoria()}
    reservas = []

    while True:
        mostrar_menu()
        opcion = input("Seleccione una opción: ")

        if opcion == "1":
            try:
                nombre = input("Nombre: ")
                identificacion = input("Identificación: ")
                correo = input("Correo: ")
                cliente = Cliente(nombre, identificacion, correo)
                clientes.append(cliente)
                print("Cliente registrado con éxito.")
            except Exception as e:
                registrar_evento(f"Error al registrar cliente: {str(e)}")
                print("Error al registrar cliente.")

        elif opcion == "2":
            print("Servicios disponibles:")
            for k, v in servicios.items():
                print(f"{k}. {v.descripcion()}")

        elif opcion == "3":
            if not clientes:
                print("Debe registrar al menos un cliente primero.")
                continue

            print("Seleccione cliente:")
            for i, c in enumerate(clientes, start=1):
                print(f"{i}. {c.mostrar_info()}")
            idx_cliente = int(input("Número de cliente: ")) - 1
            cliente = clientes[idx_cliente]

            print("Seleccione servicio:")
            for k, v in servicios.items():
                print(f"{k}. {v.descripcion()}")
            idx_servicio = input("Número de servicio: ")
            servicio = servicios.get(idx_servicio)

            duracion = int(input("Duración (horas/días/sesiones): "))
            reserva = Reserva(cliente, servicio, duracion)
            print(reserva.confirmar())
            reservas.append(reserva)

        elif opcion == "4":
            print("Saliendo del sistema...")
            break

        else:
            print("Opción inválida")

if __name__ == "__main__":
    main()

