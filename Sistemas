"""
Sistema Integral de Gestión de Clientes, Servicios y Reservas — Software FJ
Curso: Programación (213023) - Fase 4: Prácticas simuladas
Institución: Universidad Nacional Abierta y a Distancia (UNAD)
"""

import logging
from abc import ABC, abstractmethod
from functools import singledispatchmethod


# ==============================================================================
# 1. CONFIGURACIÓN DEL SISTEMA DE AUDITORÍA Y REGISTRO (LOGS)
# ==============================================================================
logging.basicConfig(
    filename="sistema_fj.log",
    filemode="w",  # Reinicia el log en cada ejecución limpia de la simulación
    level=logging.INFO,
    format="%(asctime)s - [%(levelname)s] - %(message)s",
    encoding="utf-8",
)


# ==============================================================================
# 2. JERARQUÍA DE EXCEPCIONES PERSONALIZADAS
# ==============================================================================
class ErrorSistemaFJ(Exception):
    """Excepción base para todos los errores de negocio del sistema Software FJ."""


class ErrorCliente(ErrorSistemaFJ):
    """Se lanza cuando los datos de registro o validación de un cliente son inválidos."""


class ErrorServicio(ErrorSistemaFJ):
    """Se lanza ante tarifas inválidas, falta de inventario o duraciones fuera de rango."""


class ErrorReserva(ErrorSistemaFJ):
    """Se lanza cuando falla el procesamiento o cancelación de una reserva."""


# ==============================================================================
# 3. CLASES ABSTRACTAS BASE (ABSTRACCIÓN)
# ==============================================================================
class EntidadSistema(ABC):
    """Clase abstracta raíz que define el contrato mínimo de visualización."""

    @abstractmethod
    def obtener_info(self) -> str:
        """Devuelve una representación legible de la entidad."""


class Servicio(EntidadSistema, ABC):
    """Clase abstracta base para los servicios ofrecidos por Software FJ."""

    IVA = 0.19

    def __init__(self, codigo: str, nombre: str, tarifa_base: float):
        if not codigo or not codigo.strip():
            raise ErrorServicio("El código del servicio no puede estar vacío.")
        if not nombre or not nombre.strip():
            raise ErrorServicio("El nombre del servicio no puede estar vacío.")
        self._codigo = codigo.strip().upper()
        self._nombre = nombre.strip()
        self.tarifa_base = tarifa_base  # Ejecuta validación mediante @setter

    @property
    def codigo(self) -> str:
        return self._codigo

    @property
    def nombre(self) -> str:
        return self._nombre

    @property
    def tarifa_base(self) -> float:
        return self._tarifa_base

    @tarifa_base.setter
    def tarifa_base(self, valor: float):
        if not isinstance(valor, (int, float)) or valor <= 0:
            logging.error(f"Intento de asignar tarifa inválida ({valor}) al servicio '{self._nombre}'.")
            raise ErrorServicio(f"La tarifa base debe ser mayor a cero (valor recibido: {valor}).")
        self._tarifa_base = float(valor)

    @abstractmethod
    def validar_parametros(self, cantidad_tiempo: int) -> None:
        """Valida las reglas de negocio específicas de cada tipo de servicio."""

    @abstractmethod
    def calcular_costo(self, cantidad_tiempo: int, descuento: float = 0.0) -> float:
        """Calcula el costo total aplicando reglas propias, descuentos e IVA."""


# ==============================================================================
# 4. ENTIDADES Y ESPECIALIZACIONES (ENCAPSULACIÓN, HERENCIA Y POLIMORFISMO)
# ==============================================================================
class Cliente(EntidadSistema):
    """Representa un cliente del sistema con encapsulación estricta de atributos."""

    def __init__(self, nombre: str, documento: str, correo: str):
        self.nombre = nombre
        self.documento = documento
        self.correo = correo
        logging.info(f"Cliente creado correctamente: {self.__nombre} (Doc: {self.__documento}).")

    @property
    def nombre(self) -> str:
        return self.__nombre

    @nombre.setter
    def nombre(self, valor: str):
        if not isinstance(valor, str) or len(valor.strip()) < 3:
            logging.error(f"Validación fallida en nombre de cliente: '{valor}'.")
            raise ErrorCliente("El nombre del cliente debe tener al menos 3 caracteres.")
        self.__nombre = valor.strip()

    @property
    def documento(self) -> str:
        return self.__documento

    @documento.setter
    def documento(self, valor: str):
        if not isinstance(valor, str) or not valor.strip().isdigit() or len(valor.strip()) < 5:
            logging.error(f"Validación fallida en documento de cliente: '{valor}'.")
            raise ErrorCliente("El documento debe contener únicamente dígitos (mínimo 5).")
        self.__documento = valor.strip()

    @property
    def correo(self) -> str:
        return self.__correo

    @correo.setter
    def correo(self, valor: str):
        if not isinstance(valor, str) or "@" not in valor or "." not in valor.split("@")[-1]:
            logging.error(f"Validación fallida en correo de cliente: '{valor}'.")
            raise ErrorCliente(f"El correo electrónico '{valor}' no tiene un formato válido.")
        self.__correo = valor.strip().lower()

    def obtener_info(self) -> str:
        return f"Cliente [{self.__documento}] - {self.__nombre} | Correo: {self.__correo}"


class ReservaSala(Servicio):
    """Especialización para alquiler de salas empresariales por horas (1 a 12 h)."""

    def __init__(self, codigo: str, nombre: str, tarifa_hora: float, capacidad: int):
        super().__init__(codigo, nombre, tarifa_hora)
        if capacidad <= 0:
            raise ErrorServicio("La capacidad de la sala debe ser mayor a cero.")
        self.capacidad = capacidad

    def validar_parametros(self, horas: int) -> None:
        if not isinstance(horas, int) or horas < 1 or horas > 12:
            raise ErrorServicio(
                f"Duración no permitida ({horas} h) para '{self.nombre}'. Rango válido: 1 a 12 horas."
            )

    def calcular_costo(self, horas: int, descuento: float = 0.0) -> float:
        self.validar_parametros(horas)
        subtotal = self.tarifa_base * horas
        subtotal_con_descuento = subtotal * (1 - descuento)
        return round(subtotal_con_descuento * (1 + self.IVA), 2)

    def obtener_info(self) -> str:
        return (
            f"Sala [{self.codigo}] {self.nombre} | Capacidad: {self.capacidad} pers. "
            f"| Tarifa: ${self.tarifa_base:,.0f}/hora"
        )


class AlquilerEquipo(Servicio):
    """Especialización para alquiler de equipos tecnológicos por días con control de stock."""

    SEGURO_FIJO = 15000.0

    def __init__(self, codigo: str, nombre: str, tarifa_dia: float, cantidad_disponible: int):
        super().__init__(codigo, nombre, tarifa_dia)
        if cantidad_disponible < 0:
            raise ErrorServicio("La cantidad disponible no puede ser negativa.")
        self.cantidad_disponible = cantidad_disponible

    def validar_parametros(self, dias: int) -> None:
        if self.cantidad_disponible <= 0:
            raise ErrorServicio(f"Sin unidades disponibles en inventario para '{self.nombre}'.")
        if not isinstance(dias, int) or dias < 1:
            raise ErrorServicio(f"Los días de alquiler deben ser mínimo 1 (valor recibido: {dias}).")

    def calcular_costo(self, dias: int, descuento: float = 0.0) -> float:
        self.validar_parametros(dias)
        subtotal = (self.tarifa_base * dias) * (1 - descuento) + self.SEGURO_FIJO
        return round(subtotal * (1 + self.IVA), 2)

    def obtener_info(self) -> str:
        return (
            f"Equipo [{self.codigo}] {self.nombre} | Stock: {self.cantidad_disponible} unid. "
            f"| Tarifa: ${self.tarifa_base:,.0f}/día + Seguro"
        )


class AsesoriaEspecializada(Servicio):
    """Especialización para consultoría profesional por horas (descuento del 10% si > 4 h)."""

    def __init__(self, codigo: str, nombre: str, tarifa_hora: float, especialidad: str):
        super().__init__(codigo, nombre, tarifa_hora)
        self.especialidad = especialidad.strip()

    def validar_parametros(self, horas: int) -> None:
        if not isinstance(horas, int) or horas < 1 or horas > 8:
            raise ErrorServicio(
                f"Duración no permitida ({horas} h) para asesoría. Rango válido: 1 a 8 horas."
            )

    def calcular_costo(self, horas: int, descuento: float = 0.0) -> float:
        self.validar_parametros(horas)
        descuento_aplicado = max(descuento, 0.10) if horas > 4 else descuento
        subtotal = (self.tarifa_base * horas) * (1 - descuento_aplicado)
        return round(subtotal * (1 + self.IVA), 2)

    def obtener_info(self) -> str:
        return (
            f"Asesoría [{self.codigo}] {self.nombre} ({self.especialidad}) "
            f"| Tarifa: ${self.tarifa_base:,.0f}/hora"
        )


# ==============================================================================
# 5. GESTIÓN DE RESERVAS Y CONTROLADOR PRINCIPAL (SOBRECARGA Y EXCEPCIONES)
# ==============================================================================
class Reserva(EntidadSistema):
    """Gestiona el ciclo de vida transaccional de una reserva y su inventario."""

    def __init__(self, codigo_reserva: str, cliente: Cliente, servicio: Servicio, tiempo: int):
        self.codigo_reserva = codigo_reserva.strip().upper()
        self.cliente = cliente
        self.servicio = servicio
        self.tiempo = tiempo
        self.estado = "PENDIENTE"
        self.costo_total = 0.0

    def procesar_y_confirmar(self, descuento: float = 0.0) -> None:
        """Ejecuta la reserva empleando try/except/else/finally y encadenamiento raise from."""
        try:
            self.costo_total = self.servicio.calcular_costo(self.tiempo, descuento)
        except ErrorServicio as exc:
            self.estado = "RECHAZADA"
            logging.error(
                f"Reserva {self.codigo_reserva} rechazada ({self.servicio.nombre}): {exc}"
            )
            raise ErrorReserva(
                f"No se pudo procesar la reserva {self.codigo_reserva}: {exc}"
            ) from exc
        else:
            if isinstance(self.servicio, AlquilerEquipo):
                self.servicio.cantidad_disponible -= 1
            self.estado = "CONFIRMADA"
            logging.info(
                f"Reserva {self.codigo_reserva} CONFIRMADA | Cliente: {self.cliente.nombre} "
                f"| Servicio: {self.servicio.nombre} | Total: ${self.costo_total:,.2f}"
            )
        finally:
            logging.info(
                f"Auditoría transaccional finalizada para {self.codigo_reserva} "
                f"(Estado final: {self.estado})."
            )

    def cancelar(self) -> None:
        """Cancela una reserva confirmada y devuelve el stock si corresponde a un equipo."""
        if self.estado != "CONFIRMADA":
            raise ErrorReserva(
                f"Solo se pueden cancelar reservas CONFIRMADAS (estado actual: {self.estado})."
            )
        if isinstance(self.servicio, AlquilerEquipo):
            self.servicio.cantidad_disponible += 1
        self.estado = "CANCELADA"
        logging.warning(
            f"Reserva {self.codigo_reserva} CANCELADA por solicitud. Inventario actualizado."
        )

    def obtener_info(self) -> str:
        return (
            f"Reserva [{self.codigo_reserva}] | Estado: {self.estado} | "
            f"Cliente: {self.cliente.nombre} | Servicio: {self.servicio.nombre} "
            f"| Total: ${self.costo_total:,.2f}"
        )


class SistemaSoftwareFJ:
    """Controlador central en memoria con sobrecarga de métodos mediante @singledispatchmethod."""

    def __init__(self):
        self.clientes: list[Cliente] = []
        self.servicios: list[Servicio] = []
        self.reservas: list[Reserva] = []

    def registrar_cliente(self, nombre: str, documento: str, correo: str) -> Cliente:
        cliente = Cliente(nombre, documento, correo)
        self.clientes.append(cliente)
        return cliente

    def registrar_servicio(self, servicio: Servicio) -> Servicio:
        self.servicios.append(servicio)
        logging.info(f"Servicio agregado al catálogo: {servicio.obtener_info()}")
        return servicio

    def crear_reserva(
        self, codigo_reserva: str, cliente: Cliente, servicio: Servicio, tiempo: int, descuento: float = 0.0
    ) -> Reserva:
        reserva = Reserva(codigo_reserva, cliente, servicio, tiempo)
        reserva.procesar_y_confirmar(descuento)
        self.reservas.append(reserva)
        return reserva

    # --- SOBRECARGA REAL DE MÉTODOS EN PYTHON (@singledispatchmethod) ---
    @singledispatchmethod
    def buscar_servicio(self, criterio):
        """Método sobrecargado para localizar servicios por código (str) o por posición (int)."""
        raise TypeError(f"Tipo de criterio no soportado para búsqueda: {type(criterio).__name__}")

    @buscar_servicio.register
    def _(self, criterio: str) -> Servicio:
        """Sobrecarga 1: Busca un servicio por su código identificador (str)."""
        codigo_buscado = criterio.strip().upper()
        for servicio in self.servicios:
            if servicio.codigo == codigo_buscado:
                return servicio
        raise ErrorServicio(f"No existe ningún servicio con código '{codigo_buscado}'.")

    @buscar_servicio.register
    def _(self, criterio: int) -> Servicio:
        """Sobrecarga 2: Busca un servicio por su índice en el catálogo (int)."""
        if criterio < 0 or criterio >= len(self.servicios):
            raise ErrorServicio(f"Índice de servicio fuera de rango: {criterio}.")
        return self.servicios[criterio]


# ==============================================================================
# 6. SUITE DE SIMULACIÓN DE 12 OPERACIONES (PRUEBAS INTEGRALES)
# ==============================================================================
def ejecutar_simulacion():
    sistema = SistemaSoftwareFJ()
    print("=" * 78)
    print("   INICIANDO SIMULACIÓN DE 12 OPERACIONES - SISTEMA SOFTWARE FJ (FASE 4)")
    print("=" * 78)

    # --- OPERACIÓN 1: Registro exitoso de cliente 1 ---
    try:
        c1 = sistema.registrar_cliente("Nicol Quintana", "1098765432", "nicol.quintana@unad.edu.co")
        print(f"[OP 01 - ÉXITO] {c1.obtener_info()}")
    except ErrorCliente as e:
        print(f"[OP 01 - ERROR] {e}")

    # --- OPERACIÓN 2: Registro exitoso de cliente 2 ---
    try:
        c2 = sistema.registrar_cliente("Carlos Mendoza", "1023456789", "carlos.mendoza@correo.com")
        print(f"[OP 02 - ÉXITO] {c2.obtener_info()}")
    except ErrorCliente as e:
        print(f"[OP 02 - ERROR] {e}")

    # --- OPERACIÓN 3: Registro fallido de cliente (datos inválidos) ---
    try:
        sistema.registrar_cliente("Al", "ABC12", "correo_sin_arroba")
    except ErrorCliente as e:
        print(f"[OP 03 - CONTROLADO] Error esperado en cliente: {e}")

    # --- OPERACIÓN 4: Creación exitosa de servicio Sala ---
    try:
        sala = ReservaSala("S01", "Sala de Juntas Ejecutiva", 80000, 10)
        sistema.registrar_servicio(sala)
        print(f"[OP 04 - ÉXITO] {sala.obtener_info()}")
    except ErrorServicio as e:
        print(f"[OP 04 - ERROR] {e}")

    # --- OPERACIÓN 5: Creación exitosa de servicio Equipo ---
    try:
        equipo = AlquilerEquipo("E01", "MacBook Pro M3", 120000, 1)
        sistema.registrar_servicio(equipo)
        print(f"[OP 05 - ÉXITO] {equipo.obtener_info()}")
    except ErrorServicio as e:
        print(f"[OP 05 - ERROR] {e}")

    # --- OPERACIÓN 6: Creación exitosa de servicio Asesoría ---
    try:
        asesoria = AsesoriaEspecializada("A01", "Consultoría en Inteligencia Artificial", 150000, "IA Aplicada")
        sistema.registrar_servicio(asesoria)
        print(f"[OP 06 - ÉXITO] {asesoria.obtener_info()}")
    except ErrorServicio as e:
        print(f"[OP 06 - ERROR] {e}")

    # --- OPERACIÓN 7: Creación fallida de servicio (tarifa negativa) ---
    try:
        AsesoriaEspecializada("A00", "Asesoría Cloud", -50000, "Cloud")
    except ErrorServicio as e:
        print(f"[OP 07 - CONTROLADO] Error esperado en servicio: {e}")

    # --- Demostración de sobrecarga real (@singledispatchmethod) ---
    srv_por_codigo = sistema.buscar_servicio("S01")  # Despacho por str
    srv_por_indice = sistema.buscar_servicio(1)      # Despacho por int (E01)
    print(f"[SOBRECARGA] Búsqueda por str('S01'): {srv_por_codigo.nombre} | Por int(1): {srv_por_indice.nombre}")

    # --- OPERACIÓN 8: Reserva exitosa de Sala (RES-001) ---
    try:
        r1 = sistema.crear_reserva("RES-001", c1, sala, 3)
        print(f"[OP 08 - ÉXITO] {r1.obtener_info()}")
    except ErrorReserva as e:
        print(f"[OP 08 - ERROR] {e}")

    # --- OPERACIÓN 9: Reserva exitosa de Equipo (RES-002, agota stock) ---
    try:
        r2 = sistema.crear_reserva("RES-002", c2, equipo, 2)
        print(f"[OP 09 - ÉXITO] {r2.obtener_info()} | Stock restante: {equipo.cantidad_disponible}")
    except ErrorReserva as e:
        print(f"[OP 09 - ERROR] {e}")

    # --- OPERACIÓN 10: Reserva fallida por falta de stock (RES-003) ---
    try:
        sistema.crear_reserva("RES-003", c1, equipo, 1)
    except ErrorReserva as e:
        print(f"[OP 10 - CONTROLADO] {e}")

    # --- OPERACIÓN 11: Reserva fallida por duración fuera de rango (RES-004) ---
    try:
        sistema.crear_reserva("RES-004", c2, sala, 15)
    except ErrorReserva as e:
        print(f"[OP 11 - CONTROLADO] {e}")

    # --- OPERACIÓN 12: Cancelación de reserva y recuperación de inventario ---
    try:
        r2.cancelar()
        print(f"[OP 12 - ÉXITO] {r2.obtener_info()} | Stock restaurado: {equipo.cantidad_disponible}")
    except ErrorReserva as e:
        print(f"[OP 12 - ERROR] {e}")

    print("=" * 78)
    print("   SIMULACIÓN COMPLETADA. EVIDENCIAS REGISTRADAS EN 'sistema_fj.log'")
    print("=" * 78)


if __name__ == "__main__":
    ejecutar_simulacion()

   

   
