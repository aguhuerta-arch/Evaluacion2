# Evaluacion2
## 📝 Descripción del Proyecto
Este repositorio contiene un script de Python que integra la **API de Graphhopper Directions** para calcular rutas y geolocalizar ubicaciones.

El software mejorado solicita al usuario un punto de partida, un destino y el perfil de vehículo (`car`, `bike`, `foot`), y luego imprime la **distancia**, la **duración** y las **instrucciones paso a paso** de la ruta, cumpliendo con los siguientes requisitos:
* Todos los textos de interacción con el usuario están en español.
* Los valores numéricos se muestran con un máximo de dos decimales.
* La narrativa del viaje se imprime en español.

## 🚀 Instrucciones de Ejecución
Para ejecutar este script, sigue los siguientes pasos en un entorno Python 3 con las librerías necesarias:

### 1. Requisitos
Asegúrate de que la librería `requests` esté instalada:
```bash
pip install requests

[evaluacion.py](https://github.com/user-attachments/files/22984760/evaluacion.py)

import requests
import urllib.parse
from datetime import timedelta


API_KEY = "5883e94a-7c8e-43ce-8968-758139d9aa2a"
GEOCODING_URL = "https://graphhopper.com/api/1/geocode"
ROUTING_URL = "https://graphhopper.com/api/1/route"
PERFILES_VEHICULO = ['car', 'bike', 'foot']


def format_duration(milliseconds):
    """Convierte milisegundos a formato HH:MM:SS."""
    
    seconds = milliseconds // 1000
    

    td = timedelta(seconds=seconds)
    
    
    total_seconds = int(td.total_seconds())
    
    if total_seconds < 3600: 
        return f"{td.seconds // 60:02d} minutos {td.seconds % 60:02d} segundos"
    else: 
        return f"{total_seconds // 3600:02d} horas {(total_seconds % 3600) // 60:02d} minutos"

def geocoding(location, key):
    """Obtiene la latitud y longitud de una ubicación."""
    try:
        
        params = {
            'q': location,
            'limit': 1,
            'key': key
        }
        url_final = f"{GEOCODING_URL}?{urllib.parse.urlencode(params)}"
        
        
        response = requests.get(url_final)
        response.raise_for_status() 

        json_data = response.json()

        if json_data and 'hits' in json_data and json_data['hits']:
            hit = json_data['hits'][0]
            lat = hit['point']['lat']
            lng = hit['point']['lng']
            name = hit.get('name', 'Ubicación Desconocida')
            
            return (lat, lng, name)
        else:
            return None
    except requests.exceptions.RequestException as e:
        print(f"--- ERROR DE API (Geocodificación) ---")
        print(f"Error al conectar con la API o respuesta no válida: {e}")
        return None
    except Exception as e:
        print(f"--- ERROR INESPERADO (Geocodificación) ---")
        print(f"Ocurrió un error: {e}")
        return None

def routing(start_coords, end_coords, vehicle, key):
    """Calcula la ruta entre dos coordenadas y devuelve la distancia, tiempo e instrucciones."""
    try:
        start_point = f"{start_coords[0]},{start_coords[1]}"
        end_point = f"{end_coords[0]},{end_coords[1]}"

    
        params = {
            'point': [start_point, end_point],
            'vehicle': vehicle,
            'key': key,
            'locale': 'es' 
        }
        
        
        response = requests.get(ROUTING_URL, params=params)
        response.raise_for_status()

        json_data = response.json()

        if 'paths' in json_data and json_data['paths']:
            path = json_data['paths'][0]
            
            #
            distance_km = path['distance'] / 1000.0 
            
            
            time_ms = path['time'] 
            
            instructions = path['instructions']
            
            return (distance_km, time_ms, instructions)
        else:
            print("--- ERROR DE RUTA ---")
            print("No se pudo calcular una ruta para las ubicaciones y vehículo especificados.")
            return None
    except requests.exceptions.RequestException as e:
        print(f"--- ERROR DE API (Routing) ---")
        print(f"Error al conectar con la API o respuesta no válida: {e}")
        return None

def main():
    """Bucle principal para la interacción con el usuario (A y C)."""
    print("=========================================================")
    print("        MEJORA DE SOFTWARE DE GEOLOCALIZACIÓN           ")
    print("=========================================================")

    
    if API_KEY == "":
        print("🚨 ¡ALERTA! Por favor, reemplaza 'TU_CLAVE_API_AQUI' por tu clave API real.")
        return

    while True:
        
        start_location = input("\nIngrese la ubicación de PARTIDA ('s' o 'salir' para terminar): ")
        
        
        if start_location.lower() in ['s', 'salir']:
            print("¡Gracias por usar el programa! Saliendo...")
            break

        
        end_location = input("Ingrese la ubicación de DESTINO: ")
        
        
        print(f"\nPerfiles disponibles: {', '.join(PERFILES_VEHICULO)}")
        vehicle = input("Ingrese el perfil del vehículo (ej: car, bike, foot): ").lower()

        if vehicle not in PERFILES_VEHICULO:
            print(f"⚠️ Perfil de vehículo NO válido. Usando 'car' por defecto.")
            vehicle = 'car'

        print("\n--- PASO 1: OBTENIENDO COORDENADAS ---")
        
        
        start_coords = geocoding(start_location, API_KEY)
        end_coords = geocoding(end_location, API_KEY)

        if not start_coords or not end_coords:
            print("❌ Error: No se pudieron obtener las coordenadas de una o ambas ubicaciones. Intente de nuevo.")
            continue
        
        start_name = start_coords[2]
        end_name = end_coords[2]
        
        print(f"✅ Partida: {start_name} (Lat: {start_coords[0]:.4f}, Lng: {start_coords[1]:.4f})")
        print(f"✅ Destino: {end_name} (Lat: {end_coords[0]:.4f}, Lng: {end_coords[1]:.4f})")

        print("\n--- PASO 2: CALCULANDO RUTA ---")
        
        
        route_info = routing(start_coords, end_coords, vehicle, API_KEY)

        if not route_info:
            print("❌ No se pudo calcular la ruta. Verifique los datos ingresados.")
            continue

        distance_km, time_ms, instructions = route_info
        
        
        print("\n=======================================================")
        print(f"         RUTA DE {start_name} a {end_name} en {vehicle.upper()} ")
        print("=======================================================")
        
        
        print(f"➡️ Distancia Recorrida: {distance_km:.2f} km") 
        print(f"⏱️ Duración del Viaje: {format_duration(time_ms)}")
        print("=======================================================")
        
        
        print("\n--- INSTRUCCIONES PASO A PASO (Narrativa) ---")
        if instructions:
            for i, instruction in enumerate(instructions):
                
                print(f"  {i+1}. {instruction['text']}")
        else:
            print("No se encontraron instrucciones detalladas para este viaje.")
        
        print("-------------------------------------------------------\n")

if __name__ == "__main__":
    main()
