import tkinter as tk
from tkinter import ttk, messagebox, simpledialog, filedialog
import random
import time
import threading
import math
import json
import os
import sys

# Constantes del juego
ANCHO_VENTANA = 1024
ALTO_VENTANA = 768
VERSION_JUEGO = "1.0.0"
NOMBRE_JUEGO = "La Mansión Embrujada"
CONFIG_ARCHIVO = "mansion_config.json"
GUARDADO_ARCHIVO = "mansion_guardado.json"
PUNTUACION_ARCHIVO = "mansion_puntuaciones.json"

# Colores
COLOR_NEGRO = "#000000"
COLOR_ROJO_OSCURO = "#8B0000"
COLOR_ROJO_SANGRE = "#660000"
COLOR_DORADO = "#D4AF37"
COLOR_MARRON_OSCURO = "#3A2E21"
COLOR_GRIS_OSCURO = "#222222"
COLOR_PLATA = "#C0C0C0"
COLOR_BLANCO_ANTIGUO = "#F5F5DC"

class Configuracion:
    """Clase para manejar la configuración del juego"""

    def __init__(self):
        # Valores predeterminados
        self.volumen_musica = 70
        self.volumen_efectos = 80
        self.pantalla_completa = False
        self.dificultad = "Normal"  # Fácil, Normal, Difícil, Pesadilla
        self.controles = {
            "arriba": "w",
            "abajo": "s",
            "izquierda": "a",
            "derecha": "d",
            "interactuar": "e",
            "inventario": "i",
            "linterna": "f",
            "correr": "Shift_L",
            "mapa": "m",
            "pausa": "Escape"
        }
        self.brillo = 50
        self.sensibilidad_raton = 50
        self.subtitulos = True
        self.calidad_graficos = "Media"  # Baja, Media, Alta
        self.idioma = "Español"
        
        # Cargar configuración guardada si existe
        self.cargar_configuracion()
    
    def cargar_configuracion(self):
        """Carga la configuración desde el archivo"""
        try:
            if os.path.exists(CONFIG_ARCHIVO):
                with open(CONFIG_ARCHIVO, 'r', encoding='utf-8') as f:
                    config = json.load(f)
                    
                    # Actualizar atributos
                    for key, value in config.items():
                        if hasattr(self, key):
                            setattr(self, key, value)
                    
                    print("Configuración cargada correctamente")
            else:
                print("No se encontró archivo de configuración. Usando valores predeterminados")
                self.guardar_configuracion()  # Crear archivo de configuración
        except Exception as e:
            print(f"Error al cargar configuración: {e}")
    
    def guardar_configuracion(self):
        """Guarda la configuración actual en un archivo"""
        try:
            config = {
                "volumen_musica": self.volumen_musica,
                "volumen_efectos": self.volumen_efectos,
                "pantalla_completa": self.pantalla_completa,
                "dificultad": self.dificultad,
                "controles": self.controles,
                "brillo": self.brillo,
                "sensibilidad_raton": self.sensibilidad_raton,
                "subtitulos": self.subtitulos,
                "calidad_graficos": self.calidad_graficos,
                "idioma": self.idioma
            }
            
            with open(CONFIG_ARCHIVO, 'w', encoding='utf-8') as f:
                json.dump(config, f, indent=4)
                
            print("Configuración guardada correctamente")
        except Exception as e:
            print(f"Error al guardar configuración: {e}")


class SistemaSonido:
    """Sistema de sonido simulado para el juego"""
    
    def __init__(self, configuracion):
        self.configuracion = configuracion
        self.efectos_activos = {}
        self.musica_actual = None
        self.sonidos_ambientales = []
        self.ultimo_susto = 0
        
    def reproducir_efecto(self, nombre, loop=False, volumen=None):
        """Simula reproducir un efecto de sonido"""
        vol = volumen if volumen is not None else self.configuracion.volumen_efectos
        print(f"Reproduciendo efecto: {nombre} (Volumen: {vol}%)")
        
        # En un juego real, aquí se utilizaría pygame.mixer o similar
        # Para simular, solo registramos el efecto
        self.efectos_activos[nombre] = {
            "volumen": vol,
            "loop": loop,
            "timestamp": time.time()
        }
        
        # Simular el sonido con la campana del sistema
        if nombre == "susto":
            for _ in range(3):
                self.root.bell()
                time.sleep(0.1)
        else:
            self.root.bell()
            
        # Limpiar efectos no en loop después de un tiempo
        if not loop:
            self.efectos_activos[nombre]["timer"] = threading.Timer(2.0, lambda: self.efectos_activos.pop(nombre, None))
            self.efectos_activos[nombre]["timer"].start()
    
    def reproducir_musica(self, nombre, volumen=None):
        """Simula reproducir música de fondo"""
        vol = volumen if volumen is not None else self.configuracion.volumen_musica
        print(f"Reproduciendo música: {nombre} (Volumen: {vol}%)")
        
        # Detener la música actual si hay alguna
        if self.musica_actual:
            print(f"Deteniendo música anterior: {self.musica_actual}")
        
        self.musica_actual = nombre
    
    def detener_musica(self):
        """Detiene la música actual"""
        if self.musica_actual:
            print(f"Deteniendo música: {self.musica_actual}")
            self.musica_actual = None
    
    def detener_todos_sonidos(self):
        """Detiene todos los sonidos activos"""
        for nombre, datos in list(self.efectos_activos.items()):
            if "timer" in datos:
                datos["timer"].cancel()
            self.efectos_activos.pop(nombre, None)
        
        self.detener_musica()
        print("Todos los sonidos detenidos")
    
    def reproducir_susto(self):
        """Reproduce un efecto de susto aleatorio"""
        # Verificar si ha pasado suficiente tiempo desde el último susto
        tiempo_actual = time.time()
        if tiempo_actual - self.ultimo_susto < 30:  # Mínimo 30 segundos entre sustos
            return False
            
        self.ultimo_susto = tiempo_actual
        self.reproducir_efecto("susto")
        print("¡SUSTO GENERADO!")
        return True
    
    def set_root(self, root):
        """Establece la ventana principal para poder usar la campana"""
        self.root = root


class GestorGuardado:
    """Gestiona el guardado y carga de partidas"""
    
    def __init__(self):
        self.partidas_guardadas = []
        self.cargar_partidas()
    
    def cargar_partidas(self):
        """Carga las partidas guardadas desde el archivo"""
        try:
            if os.path.exists(GUARDADO_ARCHIVO):
                with open(GUARDADO_ARCHIVO, 'r', encoding='utf-8') as f:
                    self.partidas_guardadas = json.load(f)
                print(f"Se cargaron {len(self.partidas_guardadas)} partidas guardadas")
            else:
                print("No se encontró archivo de partidas guardadas")
        except Exception as e:
            print(f"Error al cargar partidas guardadas: {e}")
    
    def guardar_partida(self, datos_partida):
        """Guarda una partida nueva o sobrescribe una existente"""
        # Añadir timestamp y versión
        datos_partida["timestamp"] = time.time()
        datos_partida["fecha"] = time.strftime("%d/%m/%Y %H:%M:%S")
        datos_partida["version"] = VERSION_JUEGO
        
        # Buscar si existe una partida con el mismo slot
        for i, partida in enumerate(self.partidas_guardadas):
            if partida.get("slot") == datos_partida.get("slot"):
                # Sobrescribir
                self.partidas_guardadas[i] = datos_partida
                self._guardar_archivo()
                return True
        
        # Nueva partida
        self.partidas_guardadas.append(datos_partida)
        self._guardar_archivo()
        return True
    
    def eliminar_partida(self, slot):
        """Elimina una partida guardada por su slot"""
        for i, partida in enumerate(self.partidas_guardadas):
            if partida.get("slot") == slot:
                del self.partidas_guardadas[i]
                self._guardar_archivo()
                return True
        return False
    
    def _guardar_archivo(self):
        """Guarda las partidas en el archivo"""
        try:
            with open(GUARDADO_ARCHIVO, 'w', encoding='utf-8') as f:
                json.dump(self.partidas_guardadas, f, indent=4)
            print(f"Partidas guardadas correctamente ({len(self.partidas_guardadas)} partidas)")
        except Exception as e:
            print(f"Error al guardar partidas: {e}")


class SistemaPuntuacion:
    """Sistema para gestionar las puntuaciones más altas"""
    
    def __init__(self):
        self.puntuaciones = []
        self.cargar_puntuaciones()
    
    def cargar_puntuaciones(self):
        """Carga las puntuaciones desde el archivo"""
        try:
            if os.path.exists(PUNTUACION_ARCHIVO):
                with open(PUNTUACION_ARCHIVO, 'r', encoding='utf-8') as f:
                    self.puntuaciones = json.load(f)
                print(f"Se cargaron {len(self.puntuaciones)} puntuaciones")
            else:
                print("No se encontró archivo de puntuaciones")
        except Exception as e:
            print(f"Error al cargar puntuaciones: {e}")
    
    def agregar_puntuacion(self, nombre, puntos, tiempo, items_encontrados, nivel_completado, dificultad):
        """Agrega una nueva puntuación"""
        nueva_puntuacion = {
            "nombre": nombre,
            "puntos": puntos,
            "tiempo": tiempo,
            "items_encontrados": items_encontrados,
            "nivel_completado": nivel_completado,
            "dificultad": dificultad,
            "fecha": time.strftime("%d/%m/%Y %H:%M:%S")
        }
        
        self.puntuaciones.append(nueva_puntuacion)
        # Ordenar por puntos (mayor a menor)
        self.puntuaciones.sort(key=lambda x: x["puntos"], reverse=True)
        
        # Mantener solo las 20 mejores
        if len(self.puntuaciones) > 20:
            self.puntuaciones = self.puntuaciones[:20]
            
        self._guardar_puntuaciones()
        return True
    
    def _guardar_puntuaciones(self):
        """Guarda las puntuaciones en el archivo"""
        try:
            with open(PUNTUACION_ARCHIVO, 'w', encoding='utf-8') as f:
                json.dump(self.puntuaciones, f, indent=4)
            print(f"Puntuaciones guardadas correctamente ({len(self.puntuaciones)} puntuaciones)")
        except Exception as e:
            print(f"Error al guardar puntuaciones: {e}")


class Item:
    """Representa un objeto que el jugador puede recoger"""
    
    def __init__(self, id, nombre, descripcion, tipo, imagen=None, propiedades=None):
        self.id = id
        self.nombre = nombre
        self.descripcion = descripcion
        self.tipo = tipo  # "clave", "arma", "util", "curacion", "coleccionable"
        self.imagen = imagen
        self.propiedades = propiedades or {}
        self.usado = False
        self.cantidad = 1
    
    def usar(self, jugador=None):
        """Utiliza el item"""
        if self.tipo == "curacion" and jugador:
            jugador.vida += self.propiedades.get("vida_restaurada", 20)
            if jugador.vida > 100:
                jugador.vida = 100
            self.usado = True
            return True
        elif self.tipo == "linterna" and jugador:
            jugador.linterna_activa = not jugador.linterna_activa
            return True
        return False
    
    def combinar(self, otro_item):
        """Combina este item con otro"""
        # Ejemplo: Batería + Linterna = Linterna cargada
        if self.id == "bateria" and otro_item.id == "linterna":
            return Item(
                "linterna_cargada",
                "Linterna Cargada",
                "Una linterna que ahora tiene batería y puede iluminar zonas oscuras.",
                "util",
                propiedades={"duracion": 100, "combinado": True}
            )
        return None
    
    def to_dict(self):
        """Convierte el item a un diccionario para guardarlo"""
        return {
            "id": self.id,
            "nombre": self.nombre,
            "descripcion": self.descripcion,
            "tipo": self.tipo,
            "propiedades": self.propiedades,
            "usado": self.usado,
            "cantidad": self.cantidad
        }
    
    @classmethod
    def from_dict(cls, data):
        """Crea un item desde un diccionario"""
        item = cls(
            data["id"],
            data["nombre"],
            data["descripcion"],
            data["tipo"],
            propiedades=data.get("propiedades", {})
        )
        item.usado = data.get("usado", False)
        item.cantidad = data.get("cantidad", 1)
        return item


class Habitacion:
    """Representa una habitación o área del juego"""
    
    def __init__(self, id, nombre, descripcion, imagen=None, items=None, conexiones=None, eventos=None):
        self.id = id
        self.nombre = nombre
        self.descripcion = descripcion
        self.imagen = imagen
        self.items = items or []
        self.conexiones = conexiones or {}  # {"norte": "habitacion_id", "este": "otra_habitacion", ...}
        self.eventos = eventos or []
        self.visitada = False
        self.iluminada = False
        self.requiere_llave = False
        self.llave_requerida = None
        self.nivel_peligro = 0  # 0-10: qué tan probable es un susto
        self.secreto_encontrado = False
    
    def obtener_descripcion(self, oscuridad=False):
        """Devuelve la descripción de la habitación, considerando si está a oscuras"""
        if oscuridad and not self.iluminada:
            return "Está demasiado oscuro para ver con claridad. Necesitas una fuente de luz."
        
        desc = self.descripcion
        if self.items:
            desc += "\n\nEn la habitación puedes ver:"
            for item in self.items:
                desc += f"\n- {item.nombre}"
        
        if self.conexiones:
            desc += "\n\nSalidas:"
            for direccion, _ in self.conexiones.items():
                desc += f"\n- {direccion.capitalize()}"
        
        return desc
    
    def agregar_item(self, item):
        """Agrega un item a la habitación"""
        self.items.append(item)
    
    def quitar_item(self, item_id):
        """Quita un item de la habitación por su ID"""
        for i, item in enumerate(self.items):
            if item.id == item_id:
                return self.items.pop(i)
        return None
    
    def agregar_evento(self, evento):
        """Agrega un evento a la habitación"""
        self.eventos.append(evento)
    
    def to_dict(self):
        """Convierte la habitación a un diccionario para guardarla"""
        return {
            "id": self.id,
            "nombre": self.nombre,
            "descripcion": self.descripcion,
            "items": [item.to_dict() for item in self.items],
            "conexiones": self.conexiones,
            "visitada": self.visitada,
            "iluminada": self.iluminada,
            "requiere_llave": self.requiere_llave,
            "llave_requerida": self.llave_requerida,
            "nivel_peligro": self.nivel_peligro,
            "secreto_encontrado": self.secreto_encontrado
        }
    
    @classmethod
    def from_dict(cls, data):
        """Crea una habitación desde un diccionario"""
        habitacion = cls(
            data["id"],
            data["nombre"],
            data["descripcion"],
            conexiones=data.get("conexiones", {})
        )
        habitacion.items = [Item.from_dict(item_data) for item_data in data.get("items", [])]
        habitacion.visitada = data.get("visitada", False)
        habitacion.iluminada = data.get("iluminada", False)
        habitacion.requiere_llave = data.get("requiere_llave", False)
        habitacion.llave_requerida = data.get("llave_requerida", None)
        habitacion.nivel_peligro = data.get("nivel_peligro", 0)
        habitacion.secreto_encontrado = data.get("secreto_encontrado", False)
        return habitacion


class Jugador:
    """Representa al jugador en el juego"""
    
    def __init__(self):
        self.vida = 100
        self.cordura = 100
        self.inventario = []
        self.ubicacion_actual = None
        self.tiempo_jugado = 0
        self.linterna_activa = False
        self.bateria_linterna = 0
        self.puntuacion = 0
        self.nivel = 1
        self.historia_visitada = []
        self.ultima_accion = None
        self.ultimo_susto = 0
        self.items_encontrados = 0
        self.secretos_descubiertos = 0
        self.sustos_recibidos = 0
    
    def agregar_item(self, item):
        """Agrega un item al inventario"""
        # Comprobar si ya tenemos un item similar para agrupar
        for inv_item in self.inventario:
            if inv_item.id == item.id and inv_item.tipo == item.tipo:
                inv_item.cantidad += 1
                return True
        
        self.inventario.append(item)
        self.items_encontrados += 1
        return True
    
    def usar_item(self, item_id):
        """Usa un item del inventario"""
        for i, item in enumerate(self.inventario):
            if item.id == item_id:
                result = item.usar(self)
                if item.usado:
                    self.inventario.pop(i)
                return result
        return False
    
    def eliminar_item(self, item_id):
        """Elimina un item del inventario"""
        for i, item in enumerate(self.inventario):
            if item.id == item_id:
                self.inventario.pop(i)
                return True
        return False
    
    def tiene_item(self, item_id):
        """Comprueba si el jugador tiene un item específico"""
        for item in self.inventario:
            if item.id == item_id:
                return True
        return False
    
    def mover_a(self, habitacion_id):
        """Mueve al jugador a una nueva habitación"""
        self.ubicacion_actual = habitacion_id
        self.historia_visitada.append(habitacion_id)
    
    def recibir_susto(self, intensidad=10):
        """El jugador recibe un susto que afecta su vida y cordura"""
        daño = random.randint(intensidad//2, intensidad)
        self.vida -= daño // 2
        self.cordura -= daño
        self.sustos_recibidos += 1
        self.ultimo_susto = time.time()
        
        if self.vida < 0:
            self.vida = 0
        if self.cordura < 0:
            self.cordura = 0
        
        return daño
    
    def actualizar_linterna(self):
        """Actualiza el estado de la linterna"""
        if self.linterna_activa and self.bateria_linterna > 0:
            self.bateria_linterna -= 1
            if self.bateria_linterna <= 0:
                self.linterna_activa = False
                self.bateria_linterna = 0
                return "Tu linterna se ha quedado sin batería"
        return None
    
    def to_dict(self):
        """Convierte el jugador a un diccionario para guardarlo"""
        return {
            "vida": self.vida,
            "cordura": self.cordura,
            "inventario": [item.to_dict() for item in self.inventario],
            "ubicacion_actual": self.ubicacion_actual,
            "tiempo_jugado": self.tiempo_jugado,
            "linterna_activa": self.linterna_activa,
            "bateria_linterna": self.bateria_linterna,
            "puntuacion": self.puntuacion,
            "nivel": self.nivel,
            "historia_visitada": self.historia_visitada,
            "items_encontrados": self.items_encontrados,
            "secretos_descubiertos": self.secretos_descubiertos,
            "sustos_recibidos": self.sustos_recibidos
        }
    
    @classmethod
    def from_dict(cls, data):
        """Crea un jugador desde un diccionario"""
        jugador = cls()
        jugador.vida = data.get("vida", 100)
        jugador.cordura = data.get("cordura", 100)
        jugador.inventario = [Item.from_dict(item_data) for item_data in data.get("inventario", [])]
        jugador.ubicacion_actual = data.get("ubicacion_actual")
        jugador.tiempo_jugado = data.get("tiempo_jugado", 0)
        jugador.linterna_activa = data.get("linterna_activa", False)
        jugador.bateria_linterna = data.get("bateria_linterna", 0)
        jugador.puntuacion = data.get("puntuacion", 0)
        jugador.nivel = data.get("nivel", 1)
        jugador.historia_visitada = data.get("historia_visitada", [])
        jugador.items_encontrados = data.get("items_encontrados", 0)
        jugador.secretos_descubiertos = data.get("secretos_descubiertos", 0)
        jugador.sustos_recibidos = data.get("sustos_recibidos", 0)
        return jugador


class Evento:
    """Representa un evento en el juego, como un susto, hallazgo o animación"""
    
    def __init__(self, tipo, mensaje, condicion=None, accion=None, probabilidad=100):
        self.tipo = tipo  # susto, descubrimiento, animacion, mensaje
        self.mensaje = mensaje
        self.condicion = condicion  # Función que devuelve True/False si se cumple la condición
        self.accion = accion  # Función que se ejecuta cuando se activa el evento
        self.probabilidad = probabilidad  # % de probabilidad de que ocurra
        self.activado = False
    
    def verificar(self, motor_juego):
        """Verifica si el evento debe activarse"""
        if self.activado:
            return False
            
        # Verificar probabilidad
        if random.randint(1, 100) > self.probabilidad:
            return False
            
        # Verificar condición si existe
        if self.condicion and not self.condicion(motor_juego):
            return False
            
        return True
    
    def activar(self, motor_juego):
        """Activa el evento"""
        self.activado = True
        
        # Ejecutar acción si existe
        if self.accion:
            self.accion(motor_juego)
            
        return self.mensaje


class GeneradorMapa:
    """Generador del mapa y contenido del juego"""
    
    def generar_mansion(self):
        """Genera el mapa de la mansión embrujada"""
        habitaciones = {}
        
        # Recibidor
        recibidor = Habitacion(
            "recibidor",
            "Recibidor de la Mansión",
            "Te encuentras en el amplio recibidor de la mansión. El lugar está cubierto de polvo y telarañas. Un gran candelabro cuelga peligrosamente del techo. La atmósfera es opresiva y sientes un escalofrío recorrer tu espalda.",
            conexiones={
                "norte": "pasillo_principal", 
                "este": "biblioteca", 
                "oeste": "sala_estar"
            }
        )
        recibidor.iluminada = True
        recibidor.nivel_peligro = 2
        
        # Sala de estar
        sala_estar = Habitacion(
            "sala_estar",
            "Sala de Estar",
            "Una habitación amplia con muebles antiguos cubiertos de sábanas polvorientas. Hay retratos en las paredes que parecen seguirte con la mirada. Una chimenea apagada domina la pared norte.",
            conexiones={
                "este": "recibidor",
                "norte": "comedor"
            }
        )
        sala_estar.nivel_peligro = 3
        
        # Añadir item linterna a la sala de estar
        linterna = Item(
            "linterna",
            "Linterna",
            "Una linterna vieja. Parece funcional pero necesita baterías.",
            "util"
        )
        sala_estar.agregar_item(linterna)
        
        # Biblioteca
        biblioteca = Habitacion(
            "biblioteca",
            "Biblioteca",
            "Estanterías llenas de libros antiguos se alzan hasta el techo. Algunos libros parecen moverse por sí solos. Hay un escritorio de madera en el centro con marcas extrañas talladas en él.",
            conexiones={
                "oeste": "recibidor",
                "norte": "estudio_secreto"
            }
        )
        biblioteca.nivel_peligro = 4
        biblioteca.requiere_llave = True
        biblioteca.llave_requerida = "llave_biblioteca"
        
        # Añadir diario a la biblioteca
        diario = Item(
            "diario",
            "Diario Antiguo",
            "Un diario viejo y desgastado. Sus páginas contienen notas sobre experimentos paranormales realizados en la mansión.",
            "coleccionable"
        )
        biblioteca.agregar_item(diario)
        
        # Pasillo Principal
        pasillo = Habitacion(
            "pasillo_principal",
            "Pasillo Principal",
            "Un largo pasillo con puertas a ambos lados. Las tablas del suelo crujen bajo tus pies. Puedes escuchar susurros provenientes de las paredes.",
            conexiones={
                "sur": "recibidor",
                "norte": "escaleras",
                "este": "estudio",
                "oeste": "comedor"
            }
        )
        pasillo.nivel_peligro = 5
        
        # Comedor
        comedor = Habitacion(
            "comedor",
            "Comedor",
            "Una amplia mesa de roble domina esta habitación. La vajilla está dispuesta como si esperara comensales, aunque cubierta de polvo. Un candelabro con velas consumidas cuelga del techo.",
            conexiones={
                "este": "pasillo_principal",
                "sur": "sala_estar",
                "norte": "cocina"
            }
        )
        comedor.nivel_peligro = 4
        
        # Añadir vela al comedor
        vela = Item(
            "vela",
            "Vela",
            "Una vela blanca parcialmente consumida. Podría ser útil en caso de oscuridad.",
            "util"
        )
        comedor.agregar_item(vela)
        
        # Cocina
        cocina = Habitacion(
            "cocina",
            "Cocina",
            "Una vieja cocina con utensilios oxidados colgando de las paredes. Hay manchas oscuras en el suelo y paredes que prefieres no examinar demasiado. Un olor nauseabundo impregna el aire.",
            conexiones={
                "sur": "comedor",
                "este": "despensa"
            }
        )
        cocina.nivel_peligro = 6
        
        # Añadir cuchillo a la cocina
        cuchillo = Item(
            "cuchillo",
            "Cuchillo de Cocina",
            "Un cuchillo de cocina oxidado pero afilado. Podría ser útil para defenderte.",
            "arma",
            propiedades={"daño": 15}
        )
        cocina.agregar_item(cuchillo)
        
        # Despensa
        despensa = Habitacion(
            "despensa",
            "Despensa",
            "Una pequeña habitación con estanterías vacías. Hay telarañas por todas partes y escuchas el sonido de ratas correteando en la oscuridad.",
            conexiones={"oeste": "cocina"
            }
        )
        despensa.nivel_peligro = 5
        
        # Añadir batería a la despensa
        bateria = Item(
            "bateria",
            "Batería",
            "Una batería que parece tener algo de carga. Podría ser útil para la linterna.",
            "util"
        )
        despensa.agregar_item(bateria)
        
        # Estudio
        estudio = Habitacion(
            "estudio",
            "Estudio",
            "Una habitación elegante con estanterías de libros y un escritorio antiguo. Hay papeles esparcidos por todas partes con símbolos extraños dibujados en ellos.",
            conexiones={
                "oeste": "pasillo_principal"
            }
        )
        estudio.nivel_peligro = 4
        
        # Añadir llave al estudio
        llave = Item(
            "llave_biblioteca",
            "Llave de la Biblioteca",
            "Una llave antigua con el símbolo de un libro grabado en ella.",
            "clave"
        )
        estudio.agregar_item(llave)
        
        # Estudio Secreto
        estudio_secreto = Habitacion(
            "estudio_secreto",
            "Estudio Secreto",
            "Un estudio oculto detrás de la biblioteca. Parece que aquí se realizaban rituales oscuros. Símbolos extraños están dibujados en el suelo y las paredes están cubiertas de inscripciones.",
            conexiones={
                "sur": "biblioteca"
            }
        )
        estudio_secreto.nivel_peligro = 7
        
        # Añadir amuleto al estudio secreto
        amuleto = Item(
            "amuleto",
            "Amuleto Protector",
            "Un amuleto antiguo con símbolos extraños. Parece tener propiedades protectoras contra lo sobrenatural.",
            "util",
            propiedades={"proteccion": 25}
        )
        estudio_secreto.agregar_item(amuleto)
        
        # Escaleras
        escaleras = Habitacion(
            "escaleras",
            "Escaleras",
            "Una imponente escalera que lleva al segundo piso. Algunos peldaños están rotos y hay manchas oscuras en la barandilla. Escuchas pasos arriba.",
            conexiones={
                "sur": "pasillo_principal",
                "arriba": "pasillo_superior"
            }
        )
        escaleras.nivel_peligro = 6
        
        # Pasillo Superior
        pasillo_superior = Habitacion(
            "pasillo_superior",
            "Pasillo Superior",
            "Un largo pasillo con puertas a ambos lados. Hay cuadros en las paredes con retratos que parecen seguirte con la mirada.",
            conexiones={
                "abajo": "escaleras",
                "este": "dormitorio_principal",
                "oeste": "habitacion_nino",
                "norte": "habitacion_invitados"
            }
        )
        pasillo_superior.nivel_peligro = 7
        
        # Dormitorio Principal
        dormitorio_principal = Habitacion(
            "dormitorio_principal",
            "Dormitorio Principal",
            "Un dormitorio amplio con una cama con dosel. Las cortinas se mueven aunque no hay brisa. Hay un espejo grande que refleja una figura que no eres tú.",
            conexiones={
                "oeste": "pasillo_superior",
                "norte": "bano"
            }
        )
        dormitorio_principal.nivel_peligro = 8
        
        # Añadir llave al dormitorio principal
        llave_atico = Item(
            "llave_atico",
            "Llave del Ático",
            "Una llave antigua con el símbolo de una casa grabado en ella.",
            "clave"
        )
        dormitorio_principal.agregar_item(llave_atico)
        
        # Baño
        bano = Habitacion(
            "bano",
            "Baño",
            "Un baño antiguo con una bañera de hierro fundido. El agua del lavabo está rojiza y el espejo está agrietado. Hay rastros de uñas en las paredes.",
            conexiones={
                "sur": "dormitorio_principal"
            }
        )
        bano.nivel_peligro = 7
        
        # Añadir botiquín al baño
        botiquin = Item(
            "botiquin",
            "Botiquín",
            "Un pequeño botiquín con algunas vendas y antiséptico.",
            "curacion",
            propiedades={"vida_restaurada": 30}
        )
        bano.agregar_item(botiquin)
        
        # Habitación de Niño
        habitacion_nino = Habitacion(
            "habitacion_nino",
            "Habitación de Niño",
            "Una habitación infantil con juguetes antiguos cubiertos de polvo. Una caja de música suena por sí sola y un caballito de madera se mece sin que nadie lo empuje.",
            conexiones={
                "este": "pasillo_superior"
            }
        )
        habitacion_nino.nivel_peligro = 9
        
        # Añadir muñeca espeluznante a la habitación del niño
        muneca = Item(
            "muneca",
            "Muñeca Espeluznante",
            "Una muñeca de porcelana con los ojos negros. Sus labios están manchados de rojo y parece susurrar algo.",
            "coleccionable"
        )
        habitacion_nino.agregar_item(muneca)
        
        # Habitación de Invitados
        habitacion_invitados = Habitacion(
            "habitacion_invitados",
            "Habitación de Invitados",
            "Una habitación simple con una cama y un armario. La ventana está tapiada y hay marcas de arañazos en la puerta, como si alguien hubiera intentado salir desesperadamente.",
            conexiones={
                "sur": "pasillo_superior",
                "este": "escalera_atico"
            }
        )
        habitacion_invitados.nivel_peligro = 6
        habitacion_invitados.requiere_llave = True
        habitacion_invitados.llave_requerida = "llave_invitados"
        
        # Escalera del Ático
        escalera_atico = Habitacion(
            "escalera_atico",
            "Escalera del Ático",
            "Una estrecha escalera que lleva al ático. Escuchas susurros y lamentos provenientes de arriba.",
            conexiones={
                "oeste": "habitacion_invitados",
                "arriba": "atico"
            }
        )
        escalera_atico.nivel_peligro = 8
        escalera_atico.requiere_llave = True
        escalera_atico.llave_requerida = "llave_atico"
        
        # Ático
        atico = Habitacion(
            "atico",
            "Ático",
            "Un amplio ático lleno de antiguos muebles y cajas. Hay un fuerte olor a azufre y sientes una presencia maligna observándote. En el centro hay un círculo ritual dibujado en el suelo.",
            conexiones={
                "abajo": "escalera_atico"
            }
        )
        atico.nivel_peligro = 10
        
        # Agregar ritual al ático
        ritual = Item(
            "libro_ritual",
            "Libro de Rituales",
            "Un libro antiguo con tapas de cuero humano. Contiene rituales para invocar y desterrar entidades del más allá.",
            "coleccionable",
            propiedades={"poder": 100}
        )
        atico.agregar_item(ritual)
        
        # Añadir las habitaciones al diccionario
        habitaciones["recibidor"] = recibidor
        habitaciones["sala_estar"] = sala_estar
        habitaciones["biblioteca"] = biblioteca
        habitaciones["pasillo_principal"] = pasillo
        habitaciones["comedor"] = comedor
        habitaciones["cocina"] = cocina
        habitaciones["despensa"] = despensa
        habitaciones["estudio"] = estudio
        habitaciones["estudio_secreto"] = estudio_secreto
        habitaciones["escaleras"] = escaleras
        habitaciones["pasillo_superior"] = pasillo_superior
        habitaciones["dormitorio_principal"] = dormitorio_principal
        habitaciones["bano"] = bano
        habitaciones["habitacion_nino"] = habitacion_nino
        habitaciones["habitacion_invitados"] = habitacion_invitados
        habitaciones["escalera_atico"] = escalera_atico
        habitaciones["atico"] = atico
        
        # Crear y asignar eventos a las habitaciones
        self._generar_eventos(habitaciones)
        
        return habitaciones
    
    def _generar_eventos(self, habitaciones):
        """Genera eventos aleatorios para las habitaciones"""
        
        # Evento para la habitación del niño
        evento_ninera = Evento(
            "susto",
            "Escuchas la voz de un niño susurrando: '¿Has visto a mi niñera? Ella está aquí... siempre está aquí...'",
            probabilidad=80
        )
        habitaciones["habitacion_nino"].agregar_evento(evento_ninera)
        
        # Evento para el baño
        evento_espejo = Evento(
            "susto",
            "El espejo agrietado muestra por un instante el reflejo de una mujer con la cara desfigurada.",
            probabilidad=70
        )
        habitaciones["bano"].agregar_evento(evento_espejo)
        
        # Evento para la biblioteca
        evento_libro = Evento(
            "descubrimiento",
            "Uno de los libros cae de la estantería. Al abrirlo, encuentras un pasaje subrayado que habla sobre un ritual para sellar entidades malignas.",
            probabilidad=60
        )
        habitaciones["biblioteca"].agregar_evento(evento_libro)
        
        # Evento para el comedor
        evento_comedor = Evento(
            "susto",
            "Las sillas se mueven solas alrededor de la mesa, como si comensales invisibles se sentaran a cenar.",
            probabilidad=65
        )
        habitaciones["comedor"].agregar_evento(evento_comedor)
        
        # Evento para el ático
        evento_atico = Evento(
            "susto",
            "Una figura oscura se materializa en el círculo ritual. Sus ojos rojos te miran fijamente antes de desvanecerse con un grito desgarrador.",
            probabilidad=90
        )
        habitaciones["atico"].agregar_evento(evento_atico)
        
        # Evento para el dormitorio principal
        evento_dormitorio = Evento(
            "susto",
            "La cama se hunde como si alguien invisible se acostara en ella. Las sábanas se mueven lentamente.",
            probabilidad=75
        )
        habitaciones["dormitorio_principal"].agregar_evento(evento_dormitorio)
        
        # Evento para las escaleras
        evento_escaleras = Evento(
            "susto",
            "Escuchas pasos pesados bajando por las escaleras, pero no hay nadie visible.",
            probabilidad=70
        )
        habitaciones["escaleras"].agregar_evento(evento_escaleras)
        
        # Evento para la cocina
        evento_cocina = Evento(
            "susto",
            "Los cuchillos en la pared tiemblan y uno de ellos cae al suelo con un ruido metálico.",
            probabilidad=65
        )
        habitaciones["cocina"].agregar_evento(evento_cocina)


class MotorJuego:
    """Motor principal del juego que maneja la lógica"""
    
    def __init__(self, configuracion):
        self.configuracion = configuracion
        self.jugador = Jugador()
        self.sistema_sonido = SistemaSonido(configuracion)
        self.gestor_guardado = GestorGuardado()
        self.sistema_puntuacion = SistemaPuntuacion()
        self.generador_mapa = GeneradorMapa()
        self.habitaciones = {}
        self.tiempo_inicio = None
        self.tiempo_pausa = 0
        self.juego_pausado = False
        self.juego_terminado = False
        self.mensaje_actual = ""
        self.historia = []  # Registro de mensajes y eventos
        self.modo_oscuridad = True
        self.timer_actualizacion = None
        self.ui = None  # Referencia a la interfaz
    
    def iniciar_nuevo_juego(self):
        """Inicia un nuevo juego"""
        self.jugador = Jugador()
        self.habitaciones = self.generador_mapa.generar_mansion()
        self.tiempo_inicio = time.time()
        self.tiempo_pausa = 0
        self.juego_pausado = False
        self.juego_terminado = False
        self.mensaje_actual = "Te despiertas en una mansión desconocida. No recuerdas cómo llegaste aquí, pero sientes una presencia maligna acechando en las sombras."
        self.historia = [self.mensaje_actual]
        
        # Colocar al jugador en el recibidor
        self.jugador.mover_a("recibidor")
        
        # Comenzar música de fondo
        self.sistema_sonido.reproducir_musica("ambiente_mansion")
        
        # Iniciar temporizador de actualización
        self._iniciar_temporizador()
        
        return True
    
    def cargar_partida(self, slot):
        """Carga una partida guardada"""
        for partida in self.gestor_guardado.partidas_guardadas:
            if partida.get("slot") == slot:
                # Cargar habitaciones
                self.habitaciones = {}
                for hab_id, hab_data in partida.get("habitaciones", {}).items():
                    self.habitaciones[hab_id] = Habitacion.from_dict(hab_data)
                
                # Cargar jugador
                self.jugador = Jugador.from_dict(partida.get("jugador", {}))
                
                # Cargar tiempos y estado
                self.tiempo_inicio = time.time() - partida.get("tiempo_jugado", 0)
                self.tiempo_pausa = 0
                self.juego_pausado = False
                self.juego_terminado = False
                
                # Cargar mensajes
                self.mensaje_actual = "Partida cargada. " + partida.get("ultimo_mensaje", "")
                self.historia = partida.get("historia", [self.mensaje_actual])
                
                # Comenzar música de fondo
                self.sistema_sonido.reproducir_musica("ambiente_mansion")
                
                # Iniciar temporizador de actualización
                self._iniciar_temporizador()
                
                return True
                
        return False
    
    def guardar_partida(self, slot, nombre=""):
        """Guarda la partida actual"""
        if not nombre:
            nombre = f"Partida {slot} - {time.strftime('%d/%m/%Y %H:%M')}"
            
        datos_partida = {
            "slot": slot,
            "nombre": nombre,
            "jugador": self.jugador.to_dict(),
            "habitaciones": {hab_id: hab.to_dict() for hab_id, hab in self.habitaciones.items()},
            "tiempo_jugado": self.obtener_tiempo_jugado(),
            "ultimo_mensaje": self.mensaje_actual,
            "historia": self.historia[-20:],  # Guardar solo los últimos 20 mensajes
            "dificultad": self.configuracion.dificultad
        }
        
        return self.gestor_guardado.guardar_partida(datos_partida)
    
    def obtener_tiempo_jugado(self):
        """Devuelve el tiempo jugado en segundos"""
        if not self.tiempo_inicio:
            return 0
            
        if self.juego_pausado:
            return self.tiempo_pausa
            
        return time.time() - self.tiempo_inicio + self.tiempo_pausa
    
    def pausar_juego(self):
        """Pausa el juego"""
        if not self.juego_pausado and not self.juego_terminado:
            self.juego_pausado = True
            self.tiempo_pausa = time.time() - self.tiempo_inicio
            
            # Pausar temporizador
            if self.timer_actualizacion:
                self.timer_actualizacion.cancel()
                
            # Pausar música
            self.sistema_sonido.detener_musica()
            self.sistema_sonido.reproducir_efecto("pausa")
            
    def reanudar_juego(self):
        """Reanuda el juego pausado"""
        if self.juego_pausado and not self.juego_terminado:
            self.juego_pausado = False
            self.tiempo_inicio = time.time() - self.tiempo_pausa
            
            # Reanudar música
            self.sistema_sonido.reproducir_musica("ambiente_mansion")
            
            # Reiniciar temporizador
            self._iniciar_temporizador()
    
    def terminar_juego(self, victoria=False):
        """Termina el juego actual"""
        if self.juego_terminado:
            return
            
        self.juego_terminado = True
        
        # Detener temporizador
        if self.timer_actualizacion:
            self.timer_actualizacion.cancel()
            
        # Calcular tiempo final y puntuación
        tiempo_total = self.obtener_tiempo_jugado()
        self.jugador.tiempo_jugado = tiempo_total
        
        # Calcular puntuación final
        puntuacion = self._calcular_puntuacion(victoria)
        self.jugador.puntuacion = puntuacion
        
        # Música final
        self.sistema_sonido.detener_todos_sonidos()
        if victoria:
            self.sistema_sonido.reproducir_musica("victoria")
            self.mensaje_actual = "¡Has logrado escapar de la mansión! Los horrores quedan atrás, pero las pesadillas te acompañarán por siempre."
        else:
            self.sistema_sonido.reproducir_musica("derrota")
            self.mensaje_actual = "La oscuridad te ha consumido. Tu cuerpo permanecerá en la mansión, y tu alma se unirá a las que ya vagan por sus pasillos."
            
        self.historia.append(self.mensaje_actual)
        
        # Mostrar puntuación
        self.mostrar_resultado(victoria)
    
    def mostrar_resultado(self, victoria):
        """Muestra el resultado final del juego"""
        tiempo_str = self._formatear_tiempo(self.jugador.tiempo_jugado)
        
        if victoria:
            titulo = "¡VICTORIA!"
            mensaje = f"Has sobrevivido a la mansión embrujada.\n\n"
        else:
            titulo = "GAME OVER"
            mensaje = f"Has sucumbido a los horrores de la mansión.\n\n"
            
        mensaje += f"Puntuación: {self.jugador.puntuacion}\n"
        mensaje += f"Tiempo jugado: {tiempo_str}\n"
        mensaje += f"Objetos encontrados: {self.jugador.items_encontrados}\n"
        mensaje += f"Secretos descubiertos: {self.jugador.secretos_descubiertos}\n"
        mensaje += f"Sustos recibidos: {self.jugador.sustos_recibidos}\n"
        
        # Mostrar diálogo de puntuación
        if self.ui:
            self.ui.mostrar_resultado(titulo, mensaje, self.jugador.puntuacion)
    
    def _formatear_tiempo(self, tiempo_segundos):
        """Formatea el tiempo en formato hh:mm:ss"""
        horas = int(tiempo_segundos // 3600)
        minutos = int((tiempo_segundos % 3600) // 60)
        segundos = int(tiempo_segundos % 60)
        return f"{horas:02d}:{minutos:02d}:{segundos:02d}"
    
    def _calcular_puntuacion(self, victoria):
        """Calcula la puntuación final"""
        puntuacion_base = 1000 if victoria else 500
        
        # Bonificación por tiempo (menos tiempo es mejor)
        tiempo_max = 3600  # 1 hora
        tiempo_actual = min(self.jugador.tiempo_jugado, tiempo_max)
        bonus_tiempo = int((1 - tiempo_actual / tiempo_max) * 500)
        
        # Bonificación por vida y cordura
        bonus_vida = int(self.jugador.vida * 2)
        bonus_cordura = int(self.jugador.cordura * 2)
        
        # Bonificación por objetos y secretos
        bonus_objetos = self.jugador.items_encontrados * 50
        bonus_secretos = self.jugador.secretos_descubiertos * 100
        
        # Penalización por sustos
        penalizacion_sustos = self.jugador.sustos_recibidos * 25
        
        # Modificador de dificultad
        mod_dificultad = {
            "Fácil": 0.8,
            "Normal": 1.0,
            "Difícil": 1.2,
            "Pesadilla": 1.5
        }.get(self.configuracion.dificultad, 1.0)
        
        # Puntuación final
        puntuacion = int((puntuacion_base + bonus_tiempo + bonus_vida + bonus_cordura + 
                         bonus_objetos + bonus_secretos - penalizacion_sustos) * mod_dificultad)
        
        return max(puntuacion, 0)  # Asegurar que no sea negativa
    
    def _iniciar_temporizador(self):
        """Inicia el temporizador de actualización del juego"""
        if self.timer_actualizacion:
            self.timer_actualizacion.cancel()
            
        self.timer_actualizacion = threading.Timer(1.0, self._actualizar_juego)
        self.timer_actualizacion.daemon = True
        self.timer_actualizacion.start()
    
    def _actualizar_juego(self):
        """Actualiza el estado del juego cada segundo"""
        if self.juego_pausado or self.juego_terminado:
            return
            
        # Actualizar estado de la linterna
        mensaje_linterna = self.jugador.actualizar_linterna()
        if mensaje_linterna:
            self.agregar_mensaje(mensaje_linterna)
            
        # Comprobar eventos aleatorios
        self._comprobar_eventos()
        
        # Posibilidad de susto aleatorio
        self._susto_aleatorio()
        
        # Actualizar interfaz
        if self.ui:
            self.ui.actualizar_interfaz()
            
        # Programar siguiente actualización
        self._iniciar_temporizador()
    
    def _comprobar_eventos(self):
        """Comprueba si debe ocurrir algún evento en la habitación actual"""
        habitacion = self.obtener_habitacion_actual()
        if not habitacion:
            return
            
        # Verificar eventos de la habitación
        for evento in habitacion.eventos:
            if not evento.activado and evento.verificar(self):
                mensaje = evento.activar(self)
                self.agregar_mensaje(mensaje)
                
                # Si es un susto, aplicar efectos
                if evento.tipo == "susto":
                    self.sistema_sonido.reproducir_susto()
                    self.jugador.recibir_susto(habitacion.nivel_peligro)
                    
                # Si es un descubrimiento, posible secreto
                elif evento.tipo == "descubrimiento" and not habitacion.secreto_encontrado:
                    habitacion.secreto_encontrado = True
                    self.jugador.secretos_descubiertos += 1
                    self.sistema_sonido.reproducir_efecto("descubrimiento")
    
    def _susto_aleatorio(self):
        """Posibilidad de generar un susto aleatorio"""
        # Verificar si ha pasado suficiente tiempo desde el último susto
        if time.time() - self.jugador.ultimo_susto < 60:  # Mínimo 1 minuto entre sustos
            return
            
        habitacion = self.obtener_habitacion_actual()
        if not habitacion:
            return
            
        # Probabilidad basada en el nivel de peligro de la habitación
        probabilidad = habitacion.nivel_peligro * 0.5  # 0-5%
        
        # Ajustar según cordura del jugador (menos cordura = más sustos)
        factor_cordura = (100 - self.jugador.cordura) / 100  # 0-1
        probabilidad *= (1 + factor_cordura)
        
        # Ajustar según dificultad
        mod_dificultad = {
            "Fácil": 0.5,
            "Normal": 1.0,
            "Difícil": 1.5,
            "Pesadilla": 2.0
        }.get(self.configuracion.dificultad, 1.0)
        probabilidad *= mod_dificultad
        
        # Intentar generar susto
        if random.random() * 100 < probabilidad:
            # Lista de posibles sustos
            sustos = [
                "Escuchas un susurro en tu oído, pero no hay nadie cerca.",
                "Las luces parpadean brevemente y crees ver una figura en las sombras.",
                "Sientes un frío repentino y ves tu aliento condensarse por un instante.",
                "Algo araña el suelo detrás de ti, pero al voltear no hay nada.",
                "Un objeto cercano cae al suelo sin razón aparente.",
                "Escuchas pasos acercándose, pero se detienen de repente.",
                "Te parece ver un rostro pálido asomarse por una puerta lejana.",
                "El aire se vuelve denso y te cuesta respirar por un momento.",
                "Un llanto infantil resuena a la distancia y luego se silencia.",
                "Una puerta se cierra de golpe en algún lugar de la mansión."
            ]
            
            mensaje = random.choice(sustos)
            self.agregar_mensaje(mensaje)
            self.sistema_sonido.reproducir_susto()
            self.jugador.recibir_susto(habitacion.nivel_peligro // 2)
    
    def mover_jugador(self, direccion):
        """Mueve al jugador en la dirección indicada"""
        habitacion_actual = self.obtener_habitacion_actual()
        if not habitacion_actual:
            return False
            
        # Verificar si la dirección es válida
        if direccion not in habitacion_actual.conexiones:
            self.agregar_mensaje(f"No puedes ir en esa dirección.")
            return False
            
        # Verificar si la habitación requiere llave
        destino_id = habitacion_actual.conexiones[direccion]
        destino = self.habitaciones.get(destino_id)
        
        if destino.requiere_llave:
            if not self.jugador.tiene_item(destino.llave_requerida):
                self.agregar_mensaje(f"La puerta está cerrada. Necesitas una llave específica.")
                return False
            else:
                self.agregar_mensaje(f"Usas la llave para abrir la puerta.")
        
        # Mover al jugador
        self.jugador.mover_a(destino_id)
        
        # Marcar como visitada
        if not destino.visitada:
            destino.visitada = True
            # Primera impresión de la habitación
            self.agregar_mensaje(f"Has entrado en {destino.nombre}.")
            self.agregar_mensaje(destino.obtener_descripcion(self.modo_oscuridad))
            
            # Efecto de sonido
            self.sistema_sonido.reproducir_efecto("nueva_habitacion")
        else:
            # Ya ha estado aquí
            self.agregar_mensaje(f"Has regresado a {destino.nombre}.")
            
        return True
    
    def recoger_item(self, item_id):
        """El jugador recoge un item de la habitación actual"""
        habitacion = self.obtener_habitacion_actual()
        if not habitacion:
            return False
            
        # Si está oscuro y no hay luz, no se puede ver para recoger
        if self.modo_oscuridad and not habitacion.iluminada and not self.jugador.linterna_activa:
            self.agregar_mensaje("Está demasiado oscuro para ver lo que intentas recoger.")
            return False
            
        # Buscar el item
        for item in habitacion.items:
            if item.id == item_id:
                # Recoger el item
                self.jugador.agregar_item(item)
                habitacion.quitar_item(item_id)
                
                self.agregar_mensaje(f"Has recogido: {item.nombre} - {item.descripcion}")
                self.sistema_sonido.reproducir_efecto("recoger_item")
                return True
                
        self.agregar_mensaje("No encuentras ese objeto en la habitación.")
        return False
    
    def usar_item(self, item_id):
        """El jugador usa un item de su inventario"""
        # Verificar si el jugador tiene el item
        if not self.jugador.tiene_item(item_id):
            self.agregar_mensaje("No tienes ese objeto en tu inventario.")
            return False
            
        # Casos especiales
        if item_id == "libro_ritual" and self.jugador.ubicacion_actual == "atico":
            self.agregar_mensaje("Comienzas a recitar el ritual antiguo. La habitación tiembla y las sombras retroceden.")
            self.agregar_mensaje("Has completado el ritual. El mal que habitaba en la mansión ha sido expulsado.")
            self.terminar_juego(victoria=True)
            return True
            
        elif item_id == "linterna" or item_id == "linterna_cargada":
            if self.jugador.bateria_linterna <= 0:
                self.agregar_mensaje("La linterna no tiene batería.")
                self.agregar_mensaje("La linterna no tiene batería.")
                return False
                
            self.jugador.linterna_activa = not self.jugador.linterna_activa
            estado = "encendido" if self.jugador.linterna_activa else "apagado"
            self.agregar_mensaje(f"Has {estado} la linterna.")
            
            # Iluminar la habitación actual si se enciende
            if self.jugador.linterna_activa:
                habitacion = self.obtener_habitacion_actual()
                if habitacion:
                    habitacion.iluminada = True
                self.sistema_sonido.reproducir_efecto("linterna_on")
            else:
                self.sistema_sonido.reproducir_efecto("linterna_off")
                
            return True
            
        elif item_id == "bateria" and self.jugador.tiene_item("linterna"):
            self.jugador.bateria_linterna = 100
            self.jugador.eliminar_item("bateria")
            self.agregar_mensaje("Has colocado la batería en la linterna. Ahora está cargada.")
            self.sistema_sonido.reproducir_efecto("cargar_bateria")
            return True
            
        elif item_id == "vela":
            habitacion = self.obtener_habitacion_actual()
            if habitacion:
                habitacion.iluminada = True
                self.agregar_mensaje("Has encendido la vela. La habitación se ilumina ligeramente.")
                self.sistema_sonido.reproducir_efecto("encender_vela")
                return True
                
        # Uso genérico del item
        result = self.jugador.usar_item(item_id)
        if result:
            self.agregar_mensaje(f"Has usado: {item_id}")
            self.sistema_sonido.reproducir_efecto("usar_item")
            return True
            
        self.agregar_mensaje("No puedes usar ese objeto aquí.")
        return False
    
    def examinar(self, objetivo=None):
        """Examina la habitación o un objeto específico"""
        habitacion = self.obtener_habitacion_actual()
        if not habitacion:
            return False
            
        # Si está oscuro y no hay luz, no se puede ver bien
        vision_limitada = self.modo_oscuridad and not habitacion.iluminada and not self.jugador.linterna_activa
            
        # Examinar la habitación
        if not objetivo or objetivo == "habitacion" or objetivo == "alrededor":
            if vision_limitada:
                self.agregar_mensaje("Está demasiado oscuro para ver con claridad. Necesitas una fuente de luz.")
                return True
                
            self.agregar_mensaje(habitacion.obtener_descripcion(False))
            
            # Lista de items en la habitación
            if habitacion.items:
                self.agregar_mensaje("Objetos en la habitación:")
                for item in habitacion.items:
                    self.agregar_mensaje(f"- {item.nombre}")
            else:
                self.agregar_mensaje("No ves nada de interés en la habitación.")
                
            return True
            
        # Examinar un objeto específico en la habitación
        for item in habitacion.items:
            if item.id == objetivo or item.nombre.lower() == objetivo.lower():
                if vision_limitada:
                    self.agregar_mensaje("Está demasiado oscuro para examinar eso con detalle.")
                    return True
                    
                self.agregar_mensaje(f"Examinas: {item.nombre}")
                self.agregar_mensaje(item.descripcion)
                return True
                
        # Examinar un objeto en el inventario
        for item in self.jugador.inventario:
            if item.id == objetivo or item.nombre.lower() == objetivo.lower():
                self.agregar_mensaje(f"Examinas: {item.nombre}")
                self.agregar_mensaje(item.descripcion)
                if item.propiedades:
                    for prop, valor in item.propiedades.items():
                        if prop not in ["combinado", "poder"]:  # Propiedades ocultas
                            self.agregar_mensaje(f"- {prop.replace('_', ' ').capitalize()}: {valor}")
                return True
                
        self.agregar_mensaje("No ves eso aquí.")
        return False
    
    def inventario(self):
        """Muestra el contenido del inventario"""
        if not self.jugador.inventario:
            self.agregar_mensaje("Tu inventario está vacío.")
            return False
            
        self.agregar_mensaje("Inventario:")
        for item in self.jugador.inventario:
            cantidad = f" (x{item.cantidad})" if item.cantidad > 1 else ""
            self.agregar_mensaje(f"- {item.nombre}{cantidad}")
            
        return True
    
    def agregar_mensaje(self, mensaje):
        """Agrega un mensaje a la historia del juego"""
        self.mensaje_actual = mensaje
        self.historia.append(mensaje)
        
        # Actualizar la interfaz si existe
        if self.ui and hasattr(self.ui, "actualizar_mensajes"):
            self.ui.actualizar_mensajes()
    
    def obtener_habitacion_actual(self):
        """Obtiene la habitación actual del jugador"""
        return self.habitaciones.get(self.jugador.ubicacion_actual)
        
    def set_ui(self, ui):
        """Establece la referencia a la interfaz de usuario"""
        self.ui = ui
        self.sistema_sonido.set_root(ui.root)


class InterfazMansion:
    """Interfaz gráfica del juego"""
    
    def __init__(self, master):
        self.root = master
        self.root.title(f"{NOMBRE_JUEGO} v{VERSION_JUEGO}")
        self.root.geometry(f"{ANCHO_VENTANA}x{ALTO_VENTANA}")
        self.root.configure(bg=COLOR_NEGRO)
        self.root.minsize(800, 600)
        
        # Cargar configuración
        self.configuracion = Configuracion()
        
        # Inicializar motor del juego
        self.motor = MotorJuego(self.configuracion)
        self.motor.set_ui(self)
        
        # Variables
        self.pantalla_actual = "menu"  # menu, juego, opciones, carga, etc.
        
        # Crear interfaz
        self._crear_interfaz()
        
        # Teclas
        self._configurar_teclas()
        
    def _crear_interfaz(self):
        """Crea los elementos de la interfaz"""
        # Marco principal
        self.marco_principal = tk.Frame(self.root, bg=COLOR_NEGRO)
        self.marco_principal.pack(fill=tk.BOTH, expand=True)
        
        # Crear las diferentes pantallas
        self._crear_menu_principal()
        self._crear_pantalla_juego()
        self._crear_pantalla_opciones()
        self._crear_pantalla_carga()
        self._crear_pantalla_creditos()
        
        # Mostrar la pantalla inicial
        self.mostrar_pantalla("menu")
    
    def _configurar_teclas(self):
        """Configura las teclas de control"""
        # Mapa de direcciones
        self.mapa_direcciones = {
            self.configuracion.controles["arriba"]: "norte",
            self.configuracion.controles["abajo"]: "sur",
            self.configuracion.controles["izquierda"]: "oeste",
            self.configuracion.controles["derecha"]: "este",
            "w": "norte",  # Alternativas
            "s": "sur",
            "a": "oeste",
            "d": "este",
            "Up": "norte",
            "Down": "sur",
            "Left": "oeste",
            "Right": "este"
        }
        
        # Vincular teclas
        self.root.bind("<KeyPress>", self._manejar_tecla)
        
        # Para pantalla completa
        self.root.bind("<F11>", self._alternar_pantalla_completa)
        self.root.bind("<Escape>", self._manejar_escape)
    
    def _manejar_tecla(self, event):
        """Maneja las pulsaciones de teclas"""
        if self.pantalla_actual != "juego" or self.motor.juego_pausado:
            return
            
        key = event.keysym.lower()
        
        # Mover jugador
        if key in self.mapa_direcciones:
            direccion = self.mapa_direcciones[key]
            self.motor.mover_jugador(direccion)
            
        # Otras teclas
        elif key == self.configuracion.controles["inventario"] or key == "i":
            self.motor.inventario()
            
        elif key == self.configuracion.controles["linterna"] or key == "f":
            if self.motor.jugador.tiene_item("linterna"):
                self.motor.usar_item("linterna")
                
        elif key == self.configuracion.controles["mapa"] or key == "m":
            # Implementar vista del mapa
            pass
            
        elif key == self.configuracion.controles["interactuar"] or key == "e":
            # Para interactuar con objetos cercanos
            self.interactuar()
    
    def _manejar_escape(self, event):
        """Maneja la tecla Escape"""
        if self.pantalla_actual == "juego":
            self._mostrar_menu_pausa()
        elif self.pantalla_actual != "menu":
            self.mostrar_pantalla("menu")
    
    def _alternar_pantalla_completa(self, event=None):
        """Alterna entre pantalla completa y ventana"""
        estado = self.root.attributes('-fullscreen')
        self.root.attributes('-fullscreen', not estado)
        self.configuracion.pantalla_completa = not estado
        self.configuracion.guardar_configuracion()
    
    def _crear_menu_principal(self):
        """Crea la pantalla del menú principal"""
        self.menu_frame = tk.Frame(self.marco_principal, bg=COLOR_NEGRO)
        
        # Título
        tk.Label(
            self.menu_frame, 
            text=NOMBRE_JUEGO, 
            font=("Arial", 36, "bold"), 
            fg=COLOR_ROJO_SANGRE, 
            bg=COLOR_NEGRO
        ).pack(pady=(50, 20))
        
        # Versión
        tk.Label(
            self.menu_frame, 
            text=f"v{VERSION_JUEGO}", 
            font=("Arial", 12), 
            fg=COLOR_DORADO, 
            bg=COLOR_NEGRO
        ).pack(pady=(0, 50))
        
        # Botones
        botones = [
            ("Nuevo Juego", lambda: self._iniciar_nuevo_juego()),
            ("Cargar Partida", lambda: self.mostrar_pantalla("carga")),
            ("Opciones", lambda: self.mostrar_pantalla("opciones")),
            ("Créditos", lambda: self.mostrar_pantalla("creditos")),
            ("Salir", lambda: self.root.destroy())
        ]
        
        for texto, comando in botones:
            btn = tk.Button(
                self.menu_frame,
                text=texto,
                font=("Arial", 16),
                command=comando,
                bg=COLOR_MARRON_OSCURO,
                fg=COLOR_DORADO,
                bd=2,
                relief=tk.RIDGE,
                padx=30,
                pady=5,
                width=15
            )
            btn.pack(pady=10)
    
    def _crear_pantalla_juego(self):
        """Crea la pantalla de juego"""
        self.juego_frame = tk.Frame(self.marco_principal, bg=COLOR_NEGRO)
        
        # Marco superior (estado del jugador)
        self.estado_frame = tk.Frame(self.juego_frame, bg=COLOR_GRIS_OSCURO, height=60)
        self.estado_frame.pack(fill=tk.X, side=tk.TOP)
        self.estado_frame.pack_propagate(False)
        
        # Vida, cordura, tiempo
        self.vida_var = tk.StringVar(value="Vida: 100%")
        self.cordura_var = tk.StringVar(value="Cordura: 100%")
        self.tiempo_var = tk.StringVar(value="Tiempo: 00:00:00")
        
        tk.Label(
            self.estado_frame, 
            textvariable=self.vida_var, 
            font=("Arial", 12),
            fg=COLOR_ROJO_OSCURO,
            bg=COLOR_GRIS_OSCURO
        ).pack(side=tk.LEFT, padx=20)
        
        tk.Label(
            self.estado_frame, 
            textvariable=self.cordura_var, 
            font=("Arial", 12),
            fg=COLOR_DORADO,
            bg=COLOR_GRIS_OSCURO
        ).pack(side=tk.LEFT, padx=20)
        
        tk.Label(
            self.estado_frame, 
            textvariable=self.tiempo_var, 
            font=("Arial", 12),
            fg=COLOR_BLANCO_ANTIGUO,
            bg=COLOR_GRIS_OSCURO
        ).pack(side=tk.RIGHT, padx=20)
        
        # Marco inferior (contenido principal)
        self.contenido_frame = tk.Frame(self.juego_frame, bg=COLOR_NEGRO)
        self.contenido_frame.pack(fill=tk.BOTH, expand=True)
        
        # Panel izquierdo (ubicación, descripción)
        self.panel_izq = tk.Frame(self.contenido_frame, bg=COLOR_GRIS_OSCURO, width=650)
        self.panel_izq.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=5, pady=5)
        
        # Nombre de ubicación
        self.ubicacion_var = tk.StringVar(value="Ubicación desconocida")
        tk.Label(
            self.panel_izq,
            textvariable=self.ubicacion_var,
            font=("Arial", 14, "bold"),
            fg=COLOR_DORADO,
            bg=COLOR_GRIS_OSCURO
        ).pack(anchor=tk.W, padx=10, pady=5)
        
        # Área de mensajes con scrollbar
        mensaje_frame = tk.Frame(self.panel_izq, bg=COLOR_GRIS_OSCURO)
        mensaje_frame.pack(fill=tk.BOTH, expand=True, padx=5, pady=5)
        
        scrollbar = tk.Scrollbar(mensaje_frame)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        
        self.mensaje_text = tk.Text(
            mensaje_frame,
            font=("Arial", 12),
            fg=COLOR_BLANCO_ANTIGUO,
            bg=COLOR_NEGRO,
            bd=2,
            padx=10,
            pady=10,
            wrap=tk.WORD,
            yscrollcommand=scrollbar.set
        )
        self.mensaje_text.pack(fill=tk.BOTH, expand=True)
        scrollbar.config(command=self.mensaje_text.yview)
        
        # Panel derecho (inventario, acciones)
        self.panel_der = tk.Frame(self.contenido_frame, bg=COLOR_GRIS_OSCURO, width=300)
        self.panel_der.pack(side=tk.RIGHT, fill=tk.BOTH, padx=5, pady=5)
        
        # Inventario
        tk.Label(
            self.panel_der,
            text="Inventario",
            font=("Arial", 14, "bold"),
            fg=COLOR_DORADO,
            bg=COLOR_GRIS_OSCURO
        ).pack(anchor=tk.W, padx=10, pady=5)
        
        # Lista de inventario con scrollbar
        inv_frame = tk.Frame(self.panel_der, bg=COLOR_GRIS_OSCURO, height=200)
        inv_frame.pack(fill=tk.X, padx=5, pady=5)
        
        scrollbar_inv = tk.Scrollbar(inv_frame)
        scrollbar_inv.pack(side=tk.RIGHT, fill=tk.Y)
        
        self.inventario_list = tk.Listbox(
            inv_frame,
            font=("Arial", 11),
            fg=COLOR_BLANCO_ANTIGUO,
            bg=COLOR_NEGRO,
            bd=2,
            height=10,
            selectbackground=COLOR_MARRON_OSCURO,
            selectforeground=COLOR_DORADO,
            yscrollcommand=scrollbar_inv.set
        )
        self.inventario_list.pack(fill=tk.BOTH, expand=True)
        scrollbar_inv.config(command=self.inventario_list.yview)
        
        # Botones de acción
        acciones_frame = tk.Frame(self.panel_der, bg=COLOR_GRIS_OSCURO)
        acciones_frame.pack(fill=tk.X, padx=5, pady=5)
        
        acciones = [
            ("Examinar", self._accion_examinar),
            ("Recoger", self._accion_recoger),
            ("Usar", self._accion_usar),
            ("Moverse", self._accion_moverse)
        ]
        
        # Crear botones en una cuadrícula 2x2
        for i, (texto, comando) in enumerate(acciones):
            fila = i // 2
            columna = i % 2
            
            btn = tk.Button(
                acciones_frame,
                text=texto,
                font=("Arial", 12),
                command=comando,
                bg=COLOR_MARRON_OSCURO,
                fg=COLOR_DORADO,
                bd=2,
                relief=tk.RIDGE,
                width=10
            )
            btn.grid(row=fila, column=columna, padx=5, pady=5, sticky="nsew")
            
        # Configurar el redimensionamiento
        acciones_frame.grid_columnconfigure(0, weight=1)
        acciones_frame.grid_columnconfigure(1, weight=1)
        
        # Área de objetos disponibles
        tk.Label(
            self.panel_der,
            text="Objetos cercanos",
            font=("Arial", 14, "bold"),
            fg=COLOR_DORADO,
            bg=COLOR_GRIS_OSCURO
        ).pack(anchor=tk.W, padx=10, pady=5)
        
        # Lista de objetos con scrollbar
        obj_frame = tk.Frame(self.panel_der, bg=COLOR_GRIS_OSCURO)
        obj_frame.pack(fill=tk.BOTH, expand=True, padx=5, pady=5)
        
        scrollbar_obj = tk.Scrollbar(obj_frame)
        scrollbar_obj.pack(side=tk.RIGHT, fill=tk.Y)
        
        self.objetos_list = tk.Listbox(
            obj_frame,
            font=("Arial", 11),
            fg=COLOR_BLANCO_ANTIGUO,
            bg=COLOR_NEGRO,
            bd=2,
            selectbackground=COLOR_MARRON_OSCURO,
            selectforeground=COLOR_DORADO,
            yscrollcommand=scrollbar_obj.set
        )
        self.objetos_list.pack(fill=tk.BOTH, expand=True)
        scrollbar_obj.config(command=self.objetos_list.yview)
        
        # Botón de menú/pausa
        tk.Button(
            self.panel_der,
            text="Menú",
            font=("Arial", 12),
            command=self._mostrar_menu_pausa,
            bg=COLOR_MARRON_OSCURO,
            fg=COLOR_DORADO,
            bd=2,
            relief=tk.RIDGE,
            width=10
        ).pack(side=tk.BOTTOM, pady=10)
    
    def _crear_pantalla_opciones(self):
        """Crea la pantalla de opciones"""
        self.opciones_frame = tk.Frame(self.marco_principal, bg=COLOR_NEGRO)
        
        # Título
        tk.Label(
            self.opciones_frame, 
            text="Opciones", 
            font=("Arial", 24, "bold"), 
            fg=COLOR_DORADO, 
            bg=COLOR_NEGRO
        ).pack(pady=(30, 20))
        
        # Marco para las opciones
        opciones_contenido = tk.Frame(self.opciones_frame, bg=COLOR_NEGRO)
        opciones_contenido.pack(fill=tk.BOTH, expand=True, padx=50)
        
        # Volumen música
        tk.Label(
            opciones_contenido,
            text="Volumen de música:",
            font=("Arial", 14),
            fg=COLOR_BLANCO_ANTIGUO,
            bg=COLOR_NEGRO
        ).grid(row=0, column=0, sticky=tk.W, pady=10)
        
        self.musica_scale = tk.Scale(
            opciones_contenido,
            from_=0,
            to=100,
            orient=tk.HORIZONTAL,
            length=200,
            bg=COLOR_GRIS_OSCURO,
            fg=COLOR_BLANCO_ANTIGUO,
            highlightthickness=0,
            troughcolor=COLOR_NEGRO
        )
        self.musica_scale.set(self.configuracion.volumen_musica)
        self.musica_scale.grid(row=0, column=1, padx=10)
        
        # Volumen efectos
        tk.Label(
            opciones_contenido,
            text="Volumen de efectos:",
            font=("Arial", 14),
            fg=COLOR_BLANCO_ANTIGUO,
            bg=COLOR_NEGRO
        ).grid(row=1, column=0, sticky=tk.W, pady=10)
        
        self.efectos_scale = tk.Scale(
            opciones_contenido,
            from_=0,
            to=100,
            orient=tk.HORIZONTAL,
            length=200,
            bg=COLOR_GRIS_OSCURO,
            fg=COLOR_BLANCO_ANTIGUO,
            highlightthickness=0,
            troughcolor=COLOR_NEGRO
        )
        self.efectos_scale.set(self.configuracion.volumen_efectos)
        self.efectos_scale.grid(row=1, column=1, padx=10)
        
        # Dificultad
        tk.Label(
            opciones_contenido,
            text="Dificultad:",
            font=("Arial", 14),
            fg=COLOR_BLANCO_ANTIGUO,
            bg=COLOR_NEGRO
        ).grid(row=2, column=0, sticky=tk.W, pady=10)
        
        self.dificultad_var = tk.StringVar(value=self.configuracion.dificultad)
        dificultad_opciones = ["Fácil", "Normal", "Difícil", "Pesadilla"]
        
        self.dificultad_combo = ttk.Combobox(
            opciones_contenido,
            textvariable=self.dificultad_var,
            values=dificultad_opciones,
            state="readonly",
            width=15,
            font=("Arial", 12)
        )
        self.dificultad_combo.grid(row=2, column=1, padx=10, sticky=tk.W)
        
        # Pantalla completa
        self.pantalla_completa_var = tk.BooleanVar(value=self.configuracion.pantalla_completa)
        tk.Checkbutton(
            opciones_contenido,
            text="Pantalla completa",
            variable=self.pantalla_completa_var,
            font=("Arial", 14),
            fg=COLOR_BLANCO_ANTIGUO,
            bg=COLOR_NEGRO,
            selectcolor=COLOR_GRIS_OSCURO,
            activebackground=COLOR_NEGRO,
            activeforeground=COLOR_BLANCO_ANTIGUO
        ).grid(row=3, column=0, columnspan=2, sticky=tk.W, pady=10)
        
        # Subtítulos
        self.subtitulos_var = tk.BooleanVar(value=self.configuracion.subtitulos)
        tk.Checkbutton(
            opciones_contenido,
            text="Mostrar subtítulos",
            variable=self.subtitulos_var,
            font=("Arial", 14),
            fg=COLOR_BLANCO_ANTIGUO,
            bg=COLOR_NEGRO,
            selectcolor=COLOR_GRIS_OSCURO,
            activebackground=COLOR_NEGRO,
            activeforeground=COLOR_BLANCO_ANTIGUO
        ).grid(row=4, column=0, columnspan=2, sticky=tk.W, pady=10)
        
        # Botones
        botones_frame = tk.Frame(self.opciones_frame, bg=COLOR_NEGRO)
        botones_frame.pack(side=tk.BOTTOM, pady=20)
        
        tk.Button(
            botones_frame,
            text="Guardar",
            font=("Arial", 14),
            command=self._guardar_opciones,
            bg=COLOR_MARRON_OSCURO,
            fg=COLOR_DORADO,
            bd=2,
            relief=tk.RIDGE,
            padx=20,
            pady=5
        ).pack(side=tk.LEFT, padx=10)
        
        tk.Button(
            botones_frame,
            text="Cancelar",
            font=("Arial", 14),
            command=lambda: self.mostrar_pantalla("menu"),
            bg=COLOR_MARRON_OSCURO,
            fg=COLOR_DORADO,
            bd=2,
            relief=tk.RIDGE,
            padx=20,
            pady=5
        ).pack(side=tk.LEFT, padx=10)
    
    def _crear_pantalla_carga(self):
        """Crea la pantalla de carga de partidas"""
        self.carga_frame = tk.Frame(self.marco_principal, bg=COLOR_NEGRO)
        
        # Título
        tk.Label(
            self.carga_frame, 
            text="Cargar Partida", 
            font=("Arial", 24, "bold"), 
            fg=COLOR_DORADO, 
            bg=COLOR_NEGRO
        ).pack(pady=(30, 20))
        
        # Marco para la lista de partidas
        partidas_frame = tk.Frame(self.carga_frame, bg=COLOR_NEGRO)
        partidas_frame.pack(fill=tk.BOTH, expand=True, padx=50, pady=10)
        
        # Lista de partidas con scrollbar
        scrollbar = tk.Scrollbar(partidas_frame)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        
        self.partidas_list = tk.Listbox(
            partidas_frame,
            font=("Arial", 12),
            fg=COLOR_BLANCO_ANTIGUO,
            bg=COLOR_GRIS_OSCURO,
            bd=2,
            selectbackground=COLOR_MARRON_OSCURO,
            selectforeground=COLOR_DORADO,
            height=10,
            width=50,
            yscrollcommand=scrollbar.set
        )
        self.partidas_list.pack(fill=tk.BOTH, expand=True)
        scrollbar.config(command=self.partidas_list.yview)
        
        # Botones
        botones_frame = tk.Frame(self.carga_frame, bg=COLOR_NEGRO)
        botones_frame.pack(side=tk.BOTTOM, pady=20)
        
        tk.Button(
            botones_frame,
            text="Cargar",
            font=("Arial", 14),
            command=self._cargar_partida_seleccionada,
            bg=COLOR_MARRON_OSCURO,
            fg=COLOR_DORADO,
            bd=2,
            relief=tk.RIDGE,
            padx=20,
            pady=5
        ).pack(side=tk.LEFT, padx=10)
        
        tk.Button(
            botones_frame,
            text="Eliminar",
            font=("Arial", 14),
            command=self._eliminar_partida_seleccionada,
            bg=COLOR_MARRON_OSCURO,
            fg=COLOR_DORADO,
            bd=2,
            relief=tk.RIDGE,
            padx=20,
            pady=5
        ).pack(side=tk.LEFT, padx=10)
        
        tk.Button(
            botones_frame,
            text="Cancelar",
            font=("Arial", 14),
            command=lambda: self.mostrar_pantalla("menu"),
            bg=COLOR_MARRON_OSCURO,
            fg=COLOR_DORADO,
            bd=2,
            relief=tk.RIDGE,
            padx=20,
            pady=5
        ).pack(side=tk.LEFT, padx=10)
    
    def _crear_pantalla_creditos(self):
        """Crea la pantalla de créditos"""
        self.creditos_frame = tk.Frame(self.marco_principal, bg=COLOR_NEGRO)
        
        # Título
        tk.Label(
            self.creditos_frame, 
            text="Créditos", 
            font=("Arial", 24, "bold"), 
            fg=COLOR_DORADO, 
            bg=COLOR_NEGRO
        ).pack(pady=(30, 20))
        
        # Marco para los créditos
        creditos_texto = tk.Frame(self.creditos_frame, bg=COLOR_NEGRO)
        creditos_texto.pack(fill=tk.BOTH, expand=True, padx=50, pady=10)
        # Texto de créditos
texto_creditos = (
    f"{NOMBRE_JUEGO} v{VERSION_JUEGO}\n\n"
    "Creado por: Diego. Puebla Cuesta"
    "Programación: Diego. Puebla Cuesta"
    "Diseño: Diego. Puebla Cuesta"
    "Historia: Diego. Puebla Cuesta"
    "© 2025 - Todos los derechos reservados\n\n"
    "¡Gracias por jugar!"
)

# Crear el Label para mostrar el texto
tk.Label(
    text=texto_creditos,  
    font=("Arial", 14),
    fg=COLOR_DORADO,
    bg=COLOR_NEGRO,
    justify=tk.LEFT
).pack(fill=tk.BOTH, expand=True)
