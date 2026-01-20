import tkinter as tk
import random
import threading
import time

# ---------------------------
# CONFIGURACIÓN DE LA VENTANA
# ---------------------------

root = tk.Tk()
root.title("Simulación DSM - Memoria Compartida Basada en Páginas")
root.geometry("1100x850")
root.configure(bg="#f5f5dc")

canvas = tk.Canvas(root, width=1080, height=400, bg="#ffffff", highlightthickness=0)
canvas.pack(padx=10, pady=(10, 5))

# ---------------------------
# ESTRUCTURAS DE DATOS
# ---------------------------

class Celda:
    def __init__(self, rect_id):
        self.rect = rect_id
        self.ocupada = False
        self.cola = []
        self.valor = 0  # Valor simulado de la celda

class Pagina:
    """Representa una página de memoria que contiene varias celdas"""
    def __init__(self, pagina_id, celdas):
        self.id = pagina_id
        self.celdas = celdas  # Lista de tuplas (nodo_id, fila, col)
        self.propietario = None  # Nodo que tiene la copia válida
        self.estado = "COMPARTIDA"  # COMPARTIDA, EXCLUSIVA, INVALIDA
        self.modificada = False
        self.copias = set()  # Nodos que tienen copia de esta página

class TablaPaginas:
    """Tabla de páginas para cada nodo"""
    def __init__(self, nodo_id):
        self.nodo_id = nodo_id
        self.paginas = {}  # {pagina_id: estado_local}
        # Estados locales: VALIDA, INVALIDA, MODIFICADA
    
    def marcar_valida(self, pagina_id):
        self.paginas[pagina_id] = "VALIDA"
    
    def marcar_invalida(self, pagina_id):
        self.paginas[pagina_id] = "INVALIDA"
    
    def marcar_modificada(self, pagina_id):
        self.paginas[pagina_id] = "MODIFICADA"
    
    def obtener_estado(self, pagina_id):
        return self.paginas.get(pagina_id, "INVALIDA")

# Estructuras globales
paginas_memoria = {}  # {pagina_id: Pagina}
tablas_paginas = {}  # {nodo_id: TablaPaginas}
escrituras_pendientes = []
sincronizando = False
pagina_counter = 0

# ---------------------------
# DIBUJAR EL CLUSTER
# ---------------------------

# Fondo degradado simulado
canvas.create_rectangle(50, 30, 1030, 380, fill="#fafbfc", outline="")
canvas.create_rectangle(50, 30, 1030, 380, outline="#2c3e50", width=2)

canvas.create_text(540, 15, text="CLUSTER DSM - MEMORIA COMPARTIDA BASADA EN PÁGINAS",
                   font=("Arial", 12, "bold"), fill="#2c3e50")

def dibujar_celdas(x_inicio, y_inicio, filas, columnas, tam=16, espacio=4):
    matriz = []
    for i in range(filas):
        fila = []
        for j in range(columnas):
            x1 = x_inicio + j * (tam + espacio)
            y1 = y_inicio + i * (tam + espacio)
            x2 = x1 + tam
            y2 = y1 + tam

            rect = canvas.create_rectangle(
                x1, y1, x2, y2,
                fill="green", outline="#1e8449", width=1.5
            )
            fila.append(Celda(rect))
        matriz.append(fila)
    return matriz

# ---------------------------
# CREACIÓN DE NODOS
# ---------------------------

nodos = {}

# Nodo 1
canvas.create_rectangle(80, 50, 250, 180, fill="#e8f4f8", outline="#3498db", width=2)
canvas.create_text(165, 62, text="Nodo 1", font=("Arial", 9, "bold"), fill="#2980b9")
nodos[1] = dibujar_celdas(95, 72, 3, 4)

# Nodo 2
canvas.create_rectangle(280, 50, 450, 180, fill="#fef5e7", outline="#f39c12", width=2)
canvas.create_text(365, 62, text="Nodo 2", font=("Arial", 9, "bold"), fill="#d68910")
nodos[2] = dibujar_celdas(295, 72, 4, 4)

# Nodo 3
canvas.create_rectangle(480, 50, 650, 180, fill="#f4ecf7", outline="#9b59b6", width=2)
canvas.create_text(565, 62, text="Nodo 3", font=("Arial", 9, "bold"), fill="#7d3c98")
nodos[3] = dibujar_celdas(495, 75, 2, 4)

# Nodo 4
canvas.create_rectangle(150, 200, 380, 360, fill="#e8f8f5", outline="#1abc9c", width=2)
canvas.create_text(265, 212, text="Nodo 4", font=("Arial", 9, "bold"), fill="#16a085")
nodos[4] = dibujar_celdas(170, 225, 5, 4)

# Nodo 5
canvas.create_rectangle(410, 200, 640, 360, fill="#fdedec", outline="#e74c3c", width=2)
canvas.create_text(525, 212, text="Nodo 5", font=("Arial", 9, "bold"), fill="#c0392b")
nodos[5] = dibujar_celdas(430, 225, 3, 4)

# ---------------------------
# INICIALIZAR PÁGINAS Y TABLAS
# ---------------------------

def inicializar_paginas():
    """Agrupa celdas en páginas (cada página = 4 celdas contiguas)"""
    global pagina_counter
    
    for nodo_id in nodos:
        tablas_paginas[nodo_id] = TablaPaginas(nodo_id)
        filas = len(nodos[nodo_id])
        columnas = len(nodos[nodo_id][0])
        
        # Crear páginas agrupando celdas de 2x2
        for fila in range(0, filas, 2):
            for col in range(0, columnas, 2):
                celdas_pagina = []
                for f in range(fila, min(fila + 2, filas)):
                    for c in range(col, min(col + 2, columnas)):
                        celdas_pagina.append((nodo_id, f, c))
                
                if celdas_pagina:
                    pagina_counter += 1
                    pagina = Pagina(pagina_counter, celdas_pagina)
                    pagina.propietario = nodo_id
                    pagina.copias.add(nodo_id)
                    paginas_memoria[pagina_counter] = pagina
                    
                    # Marcar como válida en el nodo propietario
                    tablas_paginas[nodo_id].marcar_valida(pagina_counter)

inicializar_paginas()

# ---------------------------
# PANEL DE CONTROLES
# ---------------------------

frame_controles = tk.Frame(root, bg="#ffffff", relief=tk.FLAT, bd=0)
frame_controles.pack(pady=8, fill=tk.X, padx=10)

# Contenedor interno con sombra simulada
inner_frame = tk.Frame(frame_controles, bg="#ffffff", relief=tk.RIDGE, bd=1)
inner_frame.pack(pady=2)

btn_generar = tk.Button(
    inner_frame,
    text="Generar procesos R/W",
    font=("Arial", 11, "bold"),
    bg="#27ae60",
    fg="white",
    activebackground="#229954",
    activeforeground="white",
    width=22,
    height=1,
    relief=tk.FLAT,
    cursor="hand2",
    command=lambda: None
)
btn_generar.pack(side=tk.LEFT, padx=8, pady=8)

btn_sincronizar = tk.Button(
    inner_frame,
    text="SINCRONIZAR PÁGINAS",
    font=("Arial", 11, "bold"),
    bg="#3498db",
    fg="white",
    activebackground="#2980b9",
    activeforeground="white",
    width=22,
    height=1,
    relief=tk.FLAT,
    cursor="hand2",
    command=lambda: None
)
btn_sincronizar.pack(side=tk.LEFT, padx=8, pady=8)

btn_ver_estado = tk.Button(
    inner_frame,
    text="Ver Estado Páginas",
    font=("Arial", 11, "bold"),
    bg="#f39c12",
    fg="white",
    activebackground="#d68910",
    activeforeground="white",
    width=22,
    height=1,
    relief=tk.FLAT,
    cursor="hand2",
    command=lambda: actualizar_vista_paginas()
)
btn_ver_estado.pack(side=tk.LEFT, padx=8, pady=8)

lbl_pendientes = tk.Label(
    inner_frame,
    text="Páginas modificadas: 0",
    font=("Arial", 10, "bold"),
    fg="#e74c3c",
    bg="#ffffff"
)
lbl_pendientes.pack(side=tk.LEFT, padx=15, pady=8)

# ---------------------------
# PANEL DE VISUALIZACIÓN DE PÁGINAS
# ---------------------------

frame_paginas = tk.Frame(root, bg="#ffffff", relief=tk.SOLID, bd=1)
frame_paginas.pack(pady=5, fill=tk.BOTH, padx=10, expand=False)

text_paginas = tk.Text(frame_paginas, height=12, width=130, font=("Courier", 9), 
                       bg="#f8f9fa", relief=tk.FLAT, borderwidth=0,
                       selectbackground="#3498db", selectforeground="white")
text_paginas.pack(padx=8, pady=8, fill=tk.BOTH)

def actualizar_vista_paginas():
    """Actualiza la visualización del estado de las páginas"""
    text_paginas.delete(1.0, tk.END)
    text_paginas.insert(tk.END, "PagID | Propietario | Estado Global      | Copias en Nodos\n")
    text_paginas.insert(tk.END, "─" * 85 + "\n")
    
    for pid, pag in sorted(paginas_memoria.items()):
        copias_str = ", ".join(map(str, sorted(pag.copias)))
        estado = pag.estado
        if pag.modificada:
            estado += " (MOD)"
        
        linea = f"  {pid:2d}  |   Nodo {pag.propietario}    | {estado:18s} | {copias_str}\n"
        text_paginas.insert(tk.END, linea)
    
    text_paginas.insert(tk.END, "\nTotal: {0} páginas | Cada página contiene 4 celdas (2×2)\n".format(len(paginas_memoria)))

# ---------------------------
# PANEL DE LOG
# ---------------------------

log = tk.Text(root, height=10, width=132, font=("Courier", 9),
              bg="#2c3e50", fg="#ecf0f1", insertbackground="white",
              relief=tk.FLAT, borderwidth=0,
              selectbackground="#3498db", selectforeground="white")
log.pack(pady=(5, 10), padx=10, fill=tk.BOTH, expand=True)

# ---------------------------
# FUNCIONES DE LOG
# ---------------------------

def escribir_log(mensaje):
    log.insert(tk.END, mensaje + "\n")
    log.see(tk.END)
    root.update_idletasks()

def actualizar_contador():
    paginas_mod = sum(1 for p in paginas_memoria.values() if p.modificada)
    lbl_pendientes.config(text=f"Páginas modificadas: {paginas_mod}")

# ---------------------------
# FUNCIONES AUXILIARES
# ---------------------------

def obtener_pagina_de_celda(nodo_id, fila, col):
    """Retorna la página que contiene una celda específica"""
    for pid, pag in paginas_memoria.items():
        if (nodo_id, fila, col) in pag.celdas:
            return pid, pag
    return None, None

def invalidar_copias(pagina_id, excepto_nodo=None):
    """Invalida copias de una página en otros nodos"""
    pagina = paginas_memoria[pagina_id]
    copias_invalidadas = []
    
    for nodo_id in list(pagina.copias):
        if nodo_id != excepto_nodo:
            tablas_paginas[nodo_id].marcar_invalida(pagina_id)
            copias_invalidadas.append(nodo_id)
    
    if excepto_nodo:
        pagina.copias = {excepto_nodo}
    
    return copias_invalidadas

# ---------------------------
# LÓGICA DE PROCESOS
# ---------------------------

def ejecutar_proceso(nodo_id, fila, col, tipo):
    global escrituras_pendientes
    
    celda = nodos[nodo_id][fila][col]
    pagina_id, pagina = obtener_pagina_de_celda(nodo_id, fila, col)
    
    if not pagina:
        escribir_log(f"❌ Error: Celda ({fila},{col}) no pertenece a ninguna página")
        return

    # Si está sincronizando, encolar
    if sincronizando:
        if not celda.ocupada:
            canvas.itemconfig(celda.rect, fill="yellow")
        celda.cola.append((nodo_id, fila, col, tipo))
        escribir_log(f"[ENCOLADO] Proceso {tipo} (sincronizando) en Nodo {nodo_id}, Pág {pagina_id}")
        return

    # Si está ocupada, encolar
    if celda.ocupada:
        color_actual = canvas.itemcget(celda.rect, "fill")
        if color_actual != "red":
            canvas.itemconfig(celda.rect, fill="yellow")
        celda.cola.append((nodo_id, fila, col, tipo))
        escribir_log(f"[ENCOLADO] Proceso {tipo} en Nodo {nodo_id}, Pág {pagina_id}")
        return

    # Verificar estado de la página en el nodo local
    estado_local = tablas_paginas[nodo_id].obtener_estado(pagina_id)
    
    if tipo == "W":
        # ESCRITURA: Necesita invalidar otras copias
        if estado_local == "INVALIDA":
            escribir_log(f"[FALLO PÁGINA] Nodo {nodo_id} solicita Página {pagina_id} para ESCRITURA")
            tablas_paginas[nodo_id].marcar_valida(pagina_id)
            pagina.copias.add(nodo_id)
        
        # Invalidar copias en otros nodos (protocolo de invalidación)
        copias_inv = invalidar_copias(pagina_id, excepto_nodo=nodo_id)
        if copias_inv:
            escribir_log(f"[INVALIDACIÓN] Página {pagina_id} invalidada en nodos: {copias_inv}")
        
        pagina.propietario = nodo_id
        pagina.estado = "EXCLUSIVA"
        
    else:  # LECTURA
        if estado_local == "INVALIDA":
            escribir_log(f"[FALLO PÁGINA] Nodo {nodo_id} solicita Página {pagina_id} para LECTURA")
            tablas_paginas[nodo_id].marcar_valida(pagina_id)
            pagina.copias.add(nodo_id)
            if pagina.estado == "EXCLUSIVA":
                pagina.estado = "COMPARTIDA"

    # Ocupar celda (zona crítica)
    celda.ocupada = True
    canvas.itemconfig(celda.rect, fill="red")

    tiempo = random.randint(3, 8)

    escribir_log(
        f"{'[WRITE]' if tipo == 'W' else '[READ] '} Proceso {tipo} en Nodo {nodo_id}, "
        f"Pág {pagina_id}, Celda ({fila},{col}) [{tiempo}s] - Estado: {pagina.estado}"
    )

    def trabajo():
        time.sleep(tiempo)

        def liberar():
            celda.ocupada = False
            
            if tipo == "W":
                pagina.modificada = True
                tablas_paginas[nodo_id].marcar_modificada(pagina_id)
                escrituras_pendientes.append((nodo_id, pagina_id))
                escribir_log(
                    f"[MODIFICADA] Página {pagina_id} modificada en Nodo {nodo_id} - REQUIERE SINCRONIZACIÓN"
                )
                actualizar_contador()
            
            escribir_log(f"[COMPLETADO] Proceso {tipo} en Nodo {nodo_id}, Pág {pagina_id}")

            if celda.cola:
                canvas.itemconfig(celda.rect, fill="yellow")
                siguiente = celda.cola.pop(0)
                escribir_log("➡️  Desencolando siguiente proceso...")
                root.after(500, lambda: ejecutar_proceso(*siguiente))
            else:
                canvas.itemconfig(celda.rect, fill="green")

        root.after(0, liberar)

    threading.Thread(target=trabajo, daemon=True).start()

def generar_procesos():
    if sincronizando:
        escribir_log("[ADVERTENCIA] No se pueden generar procesos durante la sincronización")
        return
        
    escribir_log("\n========== NUEVO LOTE DE PROCESOS ==========")
    for _ in range(6):
        nodo_id = random.choice(list(nodos.keys()))
        filas = len(nodos[nodo_id])
        columnas = len(nodos[nodo_id][0])
        fila = random.randint(0, filas - 1)
        col = random.randint(0, columnas - 1)
        tipo = random.choice(["R", "W"])

        ejecutar_proceso(nodo_id, fila, col, tipo)

# ---------------------------
# SINCRONIZACIÓN BASADA EN PÁGINAS
# ---------------------------

def sincronizar():
    global escrituras_pendientes, sincronizando
    
    if sincronizando:
        escribir_log("[ADVERTENCIA] Ya hay una sincronización en curso")
        return
    
    paginas_modificadas = set(pid for _, pid in escrituras_pendientes)
    
    if not paginas_modificadas:
        escribir_log("[INFO] No hay páginas modificadas para sincronizar")
        return
    
    sincronizando = True
    escribir_log("\n======== SINCRONIZACIÓN DE PÁGINAS ========")
    escribir_log(f"[SYNC] Sincronizando {len(paginas_modificadas)} páginas modificadas...")
    
    for pid in paginas_modificadas:
        pagina = paginas_memoria[pid]
        escribir_log(f"  [PÁGINA {pid}] Propagando desde Nodo {pagina.propietario} a todos los nodos")
    
    # Efecto visual
    estados_celdas = {}
    for nodo_id in nodos:
        for fila in range(len(nodos[nodo_id])):
            for col in range(len(nodos[nodo_id][fila])):
                celda = nodos[nodo_id][fila][col]
                color_actual = canvas.itemcget(celda.rect, "fill")
                estados_celdas[(nodo_id, fila, col)] = color_actual
                
                if color_actual != "red":
                    canvas.itemconfig(celda.rect, fill="cyan")
    
    def restaurar():
        global escrituras_pendientes, sincronizando
        
        # Restaurar colores y actualizar estados
        for (nodo_id, fila, col), color in estados_celdas.items():
            celda = nodos[nodo_id][fila][col]
            if canvas.itemcget(celda.rect, "fill") == "cyan":
                canvas.itemconfig(celda.rect, fill=color)
        
        # Actualizar estado de páginas
        for pid in paginas_modificadas:
            pagina = paginas_memoria[pid]
            pagina.modificada = False
            pagina.estado = "COMPARTIDA"
            
            # Validar en todos los nodos
            for nodo_id in nodos:
                tablas_paginas[nodo_id].marcar_valida(pid)
                pagina.copias.add(nodo_id)
        
        cant_sincronizadas = len(paginas_modificadas)
        escrituras_pendientes = []
        actualizar_contador()
        sincronizando = False
        
        escribir_log(f"[COMPLETADO] SINCRONIZACIÓN - {cant_sincronizadas} páginas sincronizadas")
        escribir_log("             Todas las copias de páginas ahora son VÁLIDAS y COMPARTIDAS")
        escribir_log("============================================\n")
        actualizar_vista_paginas()
    
    root.after(1500, restaurar)

# ---------------------------
# ASIGNAR COMANDOS A BOTONES
# ---------------------------

btn_generar.config(command=generar_procesos)
btn_sincronizar.config(command=sincronizar)

# ---------------------------
# MENSAJE INICIAL
# ---------------------------

escribir_log("Sistema DSM con Memoria Compartida Basada en Páginas")
escribir_log("Las celdas están agrupadas en PÁGINAS (bloques de 2x2 celdas)")
escribir_log("Protocolo: INVALIDACIÓN para escrituras, COMPARTIDA para lecturas")
escribir_log("==============================================================")
escribir_log(f"Sistema inicializado con {len(paginas_memoria)} páginas")
escribir_log("Presiona 'Generar procesos R/W' para comenzar\n")

actualizar_vista_paginas()

root.mainloop()
