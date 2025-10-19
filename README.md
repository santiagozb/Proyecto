# EC2_Kiosko_<ZamoraBarrios>
import csv
import os
import pickle
import sys
from typing import List, Dict, Tuple

CSV="datos.csv"
BIN="datos.bin"
ALERTAS="alertas.csv"

def init_demo_productos()->Tuple[List[Dict],List[List[float]]]:
    demo=[
        {"código":"A-001","nombre":"Regla","precio":"6.5","stock":"20","stock_mínimo":"5","vendidos_hoy":"0"},
        {"código":"B-100","nombre":"Lápiz","precio":"8.0","stock":"25","stock_mínimo":"6","vendidos_hoy":"0"},
        {"código":"C-200","nombre":"Borrador","precio":"3.0","stock":"30","stock_mínimo":"3","vendidos_hoy":"0"},
        {"código":"D-300","nombre":"Cuaderno_anillado","precio":"20","stock":"15","stock_mínimo":"2","vendidos_hoy":"0"},
        {"código":"F-400","nombre":"Tijera","precio":"18.0","stock":"8","stock_mínimo":"2","vendidos_hoy":"0"},
        {"código":"G-500","nombre":"Marcador","precio":"9.0","stock":"22","stock_mínimo":"5","vendidos_hoy":"0"},
        {"código":"H-600","nombre":"Resaltador","precio":"5.0","stock":"40","stock_mínimo":"6","vendidos_hoy":"0"},
        {"código":"I-700","nombre":"Calculadora","precio":"80.0","stock":"6","stock_mínimo":"2","vendidos_hoy":"0"},
        {"código":"J-800","nombre":"Archivador","precio":"10.0","stock":"13","stock_mínimo":"3","vendidos_hoy":"0"}
    ]
    ventas_semana=[[0.0 for i in range(3)] for i in range(7)]
    return demo, ventas_semana

def cargar_csv(path:str)->Tuple[List[Dict],List[List[float]]]:
    productos=[]
    try:
        with open(path,newline="",encoding="utf-8") as f:
            r=csv.DictReader(f)
            for fila in r:
                try:
                    prod={
                        "código":fila["código"].strip(),
                        "nombre":fila["nombre"].strip(),
                        "precio":float(fila.get("precio",0)),
                        "stock":int(float(fila.get("stock",0))),
                        "stock_mínimo":int(float(fila.get("stock_mínimo",0))),
                        "vendidos_hoy":int(float(fila.get("vendidos_hoy",0))) if fila.get("vendidos_hoy") not in (None,"") else 0,
                    }
                    productos.append(prod)
                except Exception:
                    continue
        ventas_semana = [[0.0 for i in range(3)] for i in range(7)]
        return productos,ventas_semana
    except FileNotFoundError:
        raise
    except Exception as e:
        print(f"Error leyendo csv:{e}")
        raise

def guardar_csv(path:str,productos:List[Dict])->None:
    with open(path,"w",newline="",encoding="utf-8") as f:
        fieldnames=["código","nombre","precio","stock","stock mínimo","vendidos hoy"]
        w=csv.DictWriter(f,fieldnames=fieldnames)
        w.writeheader()
        for p in productos:
            w.writerow({
                "código":p["código"],
                "nombre":p["nombre"],
                "precio":f"{p["precio"]:.2f}",
                "stock":p["stock"],
                "stock mínimo":p["stock mínimo"],
                "vendidos hoy":p.get("vendidos hoy",0),
            })


def guardar_bin(path:str,productos:List[Dict],ventas_semana:List[List[float]])->None:
    try:
        with open(path,"wb") as f:
            pickle.dump({"productos":productos,"ventas_semana":ventas_semana},f)
    except Exception as e:
        print(f"Error guardando binario:{e}")

def cargar_bin(path: str) -> Tuple[List[Dict], List[List[float]]]:
    try:
        with open(path, "rb") as f:
            data = pickle.load(f)
            productos = data.get("productos", [])
            ventas_semana = data.get("ventas_semana", [[0.0 for _ in range(3)] for _ in range(7)])
            return productos, ventas_semana
    except Exception:
        raise
