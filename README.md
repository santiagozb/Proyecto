# EC2_Kiosko_<ZamoraBarrios>
"""
Kiosko Universitario UCB
"""
import csv
import os
import pickle
import sys
from typing import List, Dict, Tuple

CSV_FILE = "datos.csv"
BIN_FILE = "datos.bin"
ALERTS_FILE = "alertas.csv"


def init_demo_products() -> Tuple[List[Dict], List[List[float]]]:
    demo = [
        {"codigo": "A-001", "nombre": "Regla", "precio": 6.5, "stock": 20, "stock_minimo": 5, "vendidos_hoy": 0},
        {"codigo": "B-100", "nombre": "Lapiz", "precio": 8.0, "stock": 25, "stock_minimo": 6, "vendidos_hoy": 0},
        {"codigo": "C-200", "nombre": "Borrador", "precio": 3.0, "stock": 30, "stock_minimo": 3, "vendidos_hoy": 0},
        {"codigo": "D-300", "nombre": "Cuaderno_anillado", "precio": 20.0, "stock": 15, "stock_minimo": 2, "vendidos_hoy": 0},
        {"codigo": "F-400", "nombre": "Tijera", "precio": 18.0, "stock": 8, "stock_minimo": 2, "vendidos_hoy": 0},
        {"codigo": "G-500", "nombre": "Marcador", "precio": 9.0, "stock": 22, "stock_minimo": 5, "vendidos_hoy": 0},
        {"codigo": "H-600", "nombre": "Resaltador", "precio": 5.0, "stock": 40, "stock_minimo": 6, "vendidos_hoy": 0},
        {"codigo": "I-700", "nombre": "Calculadora", "precio": 80.0, "stock": 6, "stock_minimo": 2, "vendidos_hoy": 0},
        {"codigo": "J-800", "nombre": "Archivador", "precio": 10.0, "stock": 13, "stock_minimo": 3, "vendidos_hoy": 0},
    ]
    ventas_semana = [[0.0 for _ in range(3)] for _ in range(7)]
    return demo, ventas_semana



def cargar_csv(path: str) -> Tuple[List[Dict], List[List[float]]]:
    productos = []
    try:
        with open(path, newline="", encoding="utf-8") as f:
            r = csv.DictReader(f)
            for fila in r:
                try:
                    prod = {
                        "codigo": fila["codigo"].strip(),
                        "nombre": fila["nombre"].strip(),
                        "precio": float(fila.get("precio", 0)),
                        "stock": int(float(fila.get("stock", 0))),
                        "stock_minimo": int(float(fila.get("stock_minimo", 0))),
                        "vendidos_hoy": int(float(fila.get("vendidos_hoy", 0))) if fila.get("vendidos_hoy") not in (None, "") else 0,
                    }
                    productos.append(prod)
                except Exception:
                    continue
        ventas_semana = [[0.0 for _ in range(3)] for _ in range(7)]
        return productos, ventas_semana
    except FileNotFoundError:
        raise
    except Exception as e:
        print(f"Error leyendo CSV: {e}")
        raise


def guardar_csv(path: str, productos: List[Dict]) -> None:
    with open(path, "w", newline="", encoding="utf-8") as f:
        fieldnames = ["codigo", "nombre", "precio", "stock", "stock_minimo", "vendidos_hoy"]
        w = csv.DictWriter(f, fieldnames=fieldnames)
        w.writeheader()
        for p in productos:
            w.writerow({
                "codigo": p["codigo"],
                "nombre": p["nombre"],
                "precio": f"{p['precio']:.2f}",
                "stock": p["stock"],
                "stock_minimo": p["stock_minimo"],
                "vendidos_hoy": p.get("vendidos_hoy", 0),
            })


def guardar_bin(path: str, productos: List[Dict], ventas_semana: List[List[float]]) -> None:
    try:
        with open(path, "wb") as f:
            pickle.dump({"productos": productos, "ventas_semana": ventas_semana}, f)
    except Exception as e:
        print(f"Error guardando binario: {e}")


def cargar_bin(path: str) -> Tuple[List[Dict], List[List[float]]]:
    try:
        with open(path, "rb") as f:
            data = pickle.load(f)
            productos = data.get("productos", [])
            ventas_semana = data.get("ventas_semana", [[0.0 for _ in range(3)] for _ in range(7)])
            return productos, ventas_semana
    except Exception:
        raise


def exportar_alertas_csv(path: str, productos: List[Dict]) -> None:
    bajo = [p for p in productos if p["stock"] <= p["stock_minimo"]]
    with open(path, "w", newline="", encoding="utf-8") as f:
        fieldnames = ["codigo", "nombre", "stock", "stock_minimo"]
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        for p in bajo:
            writer.writerow({
                "codigo": p["codigo"],
                "nombre": p["nombre"],
                "stock": p["stock"],
                "stock_minimo": p["stock_minimo"],
            })


# Utilidades,búsquedas


def encontrar_index_codigo(productos: List[Dict], code: str) -> int:
    for i, p in enumerate(productos):
        if p["codigo"] == code:
            return i
    return -1


def buscar_lineal_por_nombre(productos: List[Dict], nombre: str) -> List[Dict]:
    nombre_lower = nombre.lower()
    return [p for p in productos if nombre_lower in p["nombre"].lower()]


def buscar_binario_por_codigo(productos: List[Dict], codigo: str) -> int:
    low, high = 0, len(productos) - 1
    while low <= high:
        mid = (low + high) // 2
        if productos[mid]["codigo"] == codigo:
            return mid
        elif productos[mid]["codigo"] < codigo:
            low = mid + 1
        else:
            high = mid - 1
    return -1


def ordenar_burbuja(productos: List[Dict], key: str, reverse: bool = False) -> None:
    n = len(productos)
    for i in range(n - 1):
        intercambio = False
        for j in range(n - 1 - i):
            a = productos[j][key]
            b = productos[j + 1][key]
            if (not reverse and a > b) or (reverse and a < b):
                productos[j], productos[j + 1] = productos[j + 1], productos[j]
                intercambio = True
        if not intercambio:
            break


#Operaciones inventario


def anadir_producto(productos: List[Dict]) -> None:
    codigo = input("Codigo nuevo: ").strip()
    if encontrar_index_codigo(productos, codigo) != -1:
        print("Ya existe un producto con ese codigo.")
        return
    nombre = input("Nombre: ").strip()
    try:
        precio = float(input("Precio: ").strip())
        stock = int(input("Stock: ").strip())
        stock_minimo = int(input("Stock minimo: ").strip())
    except ValueError:
        print("Entrada numerica invalida. Operacion cancelada.")
        return
    prod = {"codigo": codigo, "nombre": nombre, "precio": precio, "stock": stock, "stock_minimo": stock_minimo, "vendidos_hoy": 0}
    productos.append(prod)
    print("Producto agregado con exito.")


def eliminar_producto(productos: List[Dict]) -> None:
    codigo = input("Codigo a eliminar: ").strip()
    idx = encontrar_index_codigo(productos, codigo)
    if idx == -1:
        print("Producto no encontrado")
        return
    productos.pop(idx)
    print("Producto eliminado")


def modificar_producto(productos: List[Dict]) -> None:
    codigo = input("Codigo a modificar: ").strip()
    idx = encontrar_index_codigo(productos, codigo)
    if idx == -1:
        print("Producto no encontrado.")
        return
    p = productos[idx]
    print(f"Modificando {p['codigo']} - {p['nombre']}")
    nombre = input(f"Nombre [{p['nombre']}]: ").strip() or p['nombre']
    try:
        precio_in = input(f"Precio [{p['precio']}]: ").strip()
        precio = float(precio_in) if precio_in != "" else p['precio']
        stock_in = input(f"Stock [{p['stock']}]: ").strip()
        stock = int(stock_in) if stock_in != "" else p['stock']
        stock_min_in = input(f"Stock minimo [{p['stock_minimo']}]: ").strip()
        stock_min = int(stock_min_in) if stock_min_in != "" else p['stock_minimo']
    except ValueError:
        print("Entrada numerica invalida. Modificacion cancelada.")
        return
    p.update({"nombre": nombre, "precio": precio, "stock": stock, "stock_minimo": stock_min})
    print("Producto modificado.")


def restock_producto(productos: List[Dict]) -> None:
    codigo = input("Codigo a reabastecer: ").strip()
    idx = encontrar_index_codigo(productos, codigo)
    if idx == -1:
        print("Producto no encontrado")
        return
    try:
        amount = int(input("Cantidad a sumar: ").strip())
    except ValueError:
        print("Cantidad invalida.")
        return
    if amount < 0:
        print("No puede agregar cantidad negativa.")
        return
    productos[idx]["stock"] += amount
    print("Stock actualizado.")


def vender_producto(productos: List[Dict], ventas_semana: List[List[float]]) -> None:
    codigo = input("Codigo a vender: ").strip()
    idx = encontrar_index_codigo(productos, codigo)
    if idx == -1:
        print("Producto no encontrado")
        return
    p = productos[idx]
    try:
        qt = int(input("Cantidad a vender: ").strip())
    except ValueError:
        print("Cantidad invalida")
        return
    if qt <= 0:
        print("La cantidad debe ser positiva")
        return
    if qt > p["stock"]:
        print(f"Stock insuficiente. Disponible: {p['stock']}")
        return
    try:
        dia = int(input("Dia (0=Dom,1=Lun,...,6=Sab): ").strip())
        if dia < 0 or dia > 6:
            raise ValueError
    except ValueError:
        print("Dia invalido")
        return
    try:
        horario = int(input("Horario (0=Manana,1=Tarde,2=Noche): ").strip())
        if horario < 0 or horario > 2:
            raise ValueError
    except ValueError:
        print("Horario invalido")
        return
    total = qt * p["precio"]
    p["stock"] -= qt
    p["vendidos_hoy"] = p.get("vendidos_hoy", 0) + qt
    ventas_semana[dia][horario] += total
    print(f"Venta registrada. Monto: {total:.2f}")


# Reportes


def reporte_top_3(productos: List[Dict]) -> None:
    por_ventas = sorted(productos, key=lambda x: x.get("vendidos_hoy", 0), reverse=True)
    print("Top 3 mas vendidos del dia:")
    for i, p in enumerate(por_ventas[:3], start=1):
        print(f"{i}. {p['nombre']} ({p['codigo']}) - Vendidos hoy: {p.get('vendidos_hoy', 0)}")


def reportar_low_stock(productos: List[Dict]) -> None:
    low = [p for p in productos if p["stock"] <= p["stock_minimo"]]
    if not low:
        print("No hay productos bajo stock minimo.")
        return
    print("Productos bajo stock minimo:")
    for p in low:
        print(f"{p['codigo']} - {p['nombre']} | Stock: {p['stock']} | Minimo: {p['stock_minimo']}")


def reporte_semanal(ventas_semana: List[List[float]]) -> None:
    dias = ["Dom", "Lun", "Mar", "Mie", "Jue", "Vie", "Sab"]
    print("Resumen semanal (totales por dia y horario):")
    for i, dia in enumerate(ventas_semana):
        total_dia = sum(dia)
        print(f"{dias[i]}: Manana={dia[0]:.2f} Tarde={dia[1]:.2f} Noche={dia[2]:.2f} | Total dia={total_dia:.2f}")
    total_sem = sum(sum(row) for row in ventas_semana)
    print(f"Total semanal: {total_sem:.2f}")


# Mostrar menús


def mostrar_productos(productos: List[Dict]) -> None:
    if not productos:
        print("No hay productos registrados")
        return
    print("Codigo | Nombre           | Precio  | Stock | Minimo | VendidosHoy")
    print("-" * 70)
    for p in productos:
        print(f"{p['codigo']:7} | {p['nombre'][:15]:15} | {p['precio']:7.2f} | {p['stock']:5} | {p['stock_minimo']:6} | {p.get('vendidos_hoy', 0):10}")


def ordenar_menu(productos: List[Dict]) -> None:
    print("Ordenamientos disponibles:")
    print("1) Por precio (asc)")
    print("2) Por nombre (asc)")
    print("3) Por stock (desc)")
    print("4) Ordenar por codigo (asc) - para busqueda binaria")
    opt = input("Elige opcion: ").strip()
    if opt == "1":
        ordenar_burbuja(productos, "precio", reverse=False)
        print("Ordenado por precio")
    elif opt == "2":
        ordenar_burbuja(productos, "nombre", reverse=False)
        print("Ordenado por nombre")
    elif opt == "3":
        ordenar_burbuja(productos, "stock", reverse=True)
        print("Ordenado por stock")
    elif opt == "4":
        ordenar_burbuja(productos, "codigo", reverse=False)
        print("Ordenado por codigo")
    else:
        print("Opcion invalida")


def buscar_menu(productos: List[Dict]) -> None:
    print("Busquedas:")
    print("1) Lineal por nombre")
    print("2) Binaria por codigo (requiere orden por codigo)")
    opt = input("Elige opcion: ").strip()
    if opt == "1":
        name = input("Nombre a buscar: ").strip()
        res = buscar_lineal_por_nombre(productos, name)
        if not res:
            print("No se encontraron coincidencias.")
        else:
            for p in res:
                print(f"{p['codigo']} - {p['nombre']} | Precio: {p['precio']} | Stock: {p['stock']}")
    elif opt == "2":
        code = input("Codigo a buscar: ").strip()
        idx = buscar_binario_por_codigo(productos, code)
        if idx == -1:
            print("Producto no encontrado por binaria (asegurese de ordenar por codigo primero).")
        else:
            p = productos[idx]
            print(f"Encontrado: {p['codigo']} - {p['nombre']} | Precio: {p['precio']} | Stock: {p['stock']}")
    else:
        print("Opcion invalida.")


def main_loop(productos: List[Dict], ventas_semana: List[List[float]]) -> None:
    while True:
        print("\n--- Kiosko Universitario UCB (menu) ---")
        print("1) Mostrar productos")
        print("2) Alta producto")
        print("3) Baja producto")
        print("4) Modificar producto")
        print("5) Reabastecer producto")
        print("6) Registrar venta")
        print("7) Ordenamientos")
        print("8) Busquedas")
        print("9) Reportes (Top3, Bajo stock, Resumen semanal)")
        print("10) Exportar alertas (CSV)")
        print("11) Cargar snapshot binario (datos.bin)")
        print("12) Guardar ahora (CSV + BIN)")
        print("0) Salir")
        opt = input("Elige opcion: ").strip()
        if opt == "1":
            mostrar_productos(productos)
        elif opt == "2":
            anadir_producto(productos)
        elif opt == "3":
            eliminar_producto(productos)
        elif opt == "4":
            modificar_producto(productos)
        elif opt == "5":
            restock_producto(productos)
        elif opt == "6":
            vender_producto(productos, ventas_semana)
        elif opt == "7":
            ordenar_menu(productos)
        elif opt == "8":
            buscar_menu(productos)
        elif opt == "9":
            print("Reportes disponibles:\n1) Top3  2) Bajo stock  3) Resumen semanal")
            r = input("Elige: ").strip()
            if r == "1":
                reporte_top_3(productos)
            elif r == "2":
                reportar_low_stock(productos)
            elif r == "3":
                reporte_semanal(ventas_semana)
            else:
                print("Opcion invalida.")
        elif opt == "10":
            exportar_alertas_csv(ALERTS_FILE, productos)
            print(f"Exportadas alertas a {ALERTS_FILE}")
        elif opt == "11":
            try:
                p, v = cargar_bin(BIN_FILE)
                productos.clear()
                productos.extend(p)
                for i in range(7):
                    for j in range(3):
                        ventas_semana[i][j] = v[i][j]
                print("Binario cargado con exito")
            except Exception:
                print("No se pudo cargar binario")
        elif opt == "12":
            guardar_csv(CSV_FILE, productos)
            guardar_bin(BIN_FILE, productos, ventas_semana)
            print("Guardado CSV y BIN")
        elif opt == "0":
            print("Guardando antes de salir...")
            guardar_csv(CSV_FILE, productos)
            guardar_bin(BIN_FILE, productos, ventas_semana)
            print("Guardado completo. Hasta luego.")
            break
        else:
            print("Opcion invalida. Intenta de nuevo.")


def main() -> None:
    if os.path.exists(BIN_FILE):
        try:
            productos, ventas_semana = cargar_bin(BIN_FILE)
            print(f"Cargado binario: {len(productos)} productos.")
        except Exception:
            print("Error al cargar binario, intentando CSV...")
            try:
                productos, ventas_semana = cargar_csv(CSV_FILE)
                print(f"Cargado CSV: {len(productos)} productos.")
            except FileNotFoundError:
                productos, ventas_semana = init_demo_products()
                print("CSV no encontrado. Creando demo de productos.")
    else:
        if os.path.exists(CSV_FILE):
            try:
                productos, ventas_semana = cargar_csv(CSV_FILE)
                print(f"Cargado CSV: {len(productos)} productos.")
            except Exception:
                productos, ventas_semana = init_demo_products()
                print("Error leyendo CSV. Creando demo.")
        else:
            productos, ventas_semana = init_demo_products()
            print("Archivos no encontrados. Creando demo de productos.")

    try:
        main_loop(productos, ventas_semana)
    except KeyboardInterrupt:
        # Guardado seguro en caso de Ctrl+C
        print("\nInterrupcion por teclado. Guardando estado...")
        try:
            guardar_csv(CSV_FILE, productos)
            guardar_bin(BIN_FILE, productos, ventas_semana)
            print("Guardado completo. Adios.")
        except Exception:
            print("No se pudo guardar completamente.")
        sys.exit(0)


